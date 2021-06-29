# 随机分布

为了最大程度地提供生成随机值的灵活性，
Rand 定义了 [`Distribution`] trait ：

```rust
// a producer of data of type T:
pub trait Distribution<T> {
    // the key function:
    fn sample<R: Rng + ?Sized>(&self, rng: &mut R) -> T;

    // a convenience function defined using sample:
    fn sample_iter<R>(self, rng: R) -> DistIter<Self, R, T>
    where
        Self: Sized,
        R: Rng,
    { ... }
}
```

Rand 提供了许多分布的实现；这里会谈论最常见的一些分布。
完整的细节请参考 [`distributions`] 模块和 [`rand_distr`] crate 。

# 均匀分布

最显眼的一种分布类型已经被讨论过了：
它没有模式，每个值或者一列值都是等可能的。
这被称为 **均匀分布** (uniform) 。

## 形式

Rand 实际上有几种不同形式的均匀分布，代表了不同的范围：

-   [`Standard`] 不需要参数，它根据类型进行均匀抽样。
    [`Rng::gen`] 提供了抽取这个分布的捷径。
-   [`Uniform`] 具有两种参数化形式： `Uniform::new(low, high)` （左闭右开）；
    `Uniform::new_inclusive(low, high)` （左右都是闭区间）。
    它从区间范围内抽样。

    [`Rng::gen_range`] 是定义在 [`Uniform::sample_single`] 之上的，它经过优化来生成单样本。
-   [`Alphanumeric`] 在 `0-9A-Za-z` 范围内的 `char` 类型进行均匀抽样。
-   [`Open01`] 和 [`OpenClosed01`] 提供对浮点类型均匀抽样的替代方式。（见下文）

## 根据类型均匀抽样

接下来按照类型认真讨论均匀分布：

-   对于 `bool`： [`Standard`] 对每个值以 50% 的概率抽样。
-   对于 `Option<T>`： [`Standard`] 对 `None` 值以 50% 概率抽样，
    对 `Some(value)` 则依照 `value` 的类型进行抽样。
-   对于整数（从 `u8` 到 `u128`、`usize` 和 `i*` 之类），
    [`Standard`] 从所有可能的值中抽样；
    [`Uniform`] 从参数的范围内抽样。
-   对于 `NonZeroU8` 和其他非零类型， [`Standard`] 从所有非零值中抽样（拒绝采样的方式）
-   对于 `Wrapping<T>` 形式的整数类型，抽样方式与对应的整数类型一样，以 [`Standard`] 方式抽样。
-   对于浮点数（如 `f32` 、 `f64`）：

    -   [`Standard`] 从 `[0, 1)` 范围内，分别对 `f32` 或 `f64` 类型以 24 或 53 bits 的精度抽样。
    -   [`OpenClosed01`] 从 `(0, 1]` 范围内以 24 或 53 bits 的精度抽样。
    -   [`Open01`] 从 `(0, 1)` 范围内以 23 或 52 bits 的精度抽样。
    -   [`Uniform`] 从给定范围内以 23 或 52 bits 的精度抽样。

-   对于 `char` 类型， [`Standard`] 从所有可用的 Unicode 码位 (code points) 中抽样；
    大多值因为字体支持程度可能打印不出来。
    [`Alphanumeric`] 只从 a-z, A-Z 和 0-9 中抽样。
-   对于元组和数组，元素是以上的类型时，则按照相应规则抽样。
    [`Standard`] 和 [`Uniform`] 都支持最多 12 个元素的元组和最多 32 个元素的数组，
    而且支持空元组 `()` 和空数组 `[]` 。

    如果使用的 `rustc` 大于等于 1.51 版本，开启 `min_const_gen` feature 能支持超过
    32 个元素的数组。

-   对于 SIMD 类型， [`Standard`] 和 [`Uniform`] 方式把每个元素按照上面的方式抽样。
    使用 [`Uniform`] 时， `low` 和 `high` 参数是 SIMD 类型时可以在多个范围内同时抽样。
    需要开启 [`simd_support`] feature 来支持 SIMD 功能。

[`simd_support`]:https://github.com/rust-random/rand#crate-features

# 非均匀分布

所有非均匀分布分成以下两类：连续型和离散型。
部分离散分布、所有连续分布已从 [`rand`] crate 移入专门的 [`rand_distr`] crate 。

> 译者注：这个分类是从 Rand 库的设计上阐述的，没有完全对照概率统计领域的分类。

## 离散型分布

离散分布从 bool 或整数类型上抽样。
他们既可以像上面那样进行均匀分布，又可以像下面谈到的进行非均匀分布。

一个离散分布也可能直接从一组离散的值中抽样，比如切片或枚举体。
涉及切片和迭代器类型的抽样内容，请参考 [随机序列][Sequences] 一章。
Rand 不直接提供从枚举体抽样，但提供了 `Option` 这个枚举体抽样
（[见上文](./guide-dist.html#根据类型均匀抽样)）。

### 对 bool 抽样

[`Bernoulli`] 分布已给定概率 `p` 从 bool 类型中产生随机值 `true` 。 
它可以以 `success : failure` 比例定义 `true` 的概率。
通常这个分布描述一次以成功概率 `p` 试验 (trial) 。

[`Rng::gen_bool`] 和 [`Rng::gen_ratio`] 函数是调用这个分布的捷径。

### 对整数抽样

[`Binomial`] 分布与 [`Bernoulli`] 有关，它用对 `n` 个独立试验建立模型，
其中每个试验以 `p` 概率成功，然后对成功次数计数。

注意，如果 `n` 较大， [`Binomial`] 比单次重复抽取 `n` 次试验要快得多。

[`Poisson`] 分布描述了给定 λ 参数的情况下，在固定区间内事件预期发生的次数。
由于概率计算中使用 `Float` ，所以 [`Poisson`] 的抽样结果是 `Float` 。
使用者需要整数类型时，可以对结果进行可能损失精度或造成 panic 的转换。
比如使用 `rng.sample(Poisson) as u64` 来获得 `u64` 类型的值。

注意，使用 `as` 把浮点数转换整数时，若超出范围，
则在 Rust < 1.45 的版本中导致未定义行为 (undefined behavior) ，
在 Rust >= 1.45 的版本中导致饱和转换 (saturating conversion) 。

### 加权序列

[`WeightedIndex`] 从一个权重序列中对索引抽样。
若需要直接对切片元素抽样，参阅 [随机序列][Sequences] 一章。

举例来说，加权抽样可用于建立以下模型：
桶里有一些颜色的弹珠，绿色的 5 个、红色的 15 个、蓝色的 80 个，然后从桶里抽取一个弹珠。

Rand 库目前只实现 **有放回** 抽样，即从同一个分布中重复抽样。
所以任何抽取过的弹珠都被放回桶里。
另一种抽样方式是无放回抽样，参考 [issue 596](https://github.com/rust-random/rand/issues/596) 。

也要注意，[`WeightedIndex`] 有两种实现：
[`WeightedIndex`] 经过优化适用于少量样本抽样；
而 [`alias_method::WeightedIndex`] 经过优化适用于大样本抽样。
根据基准测试的结果，建议样本量超过 1000 时使用大样本抽样。

## 连续型分布

连续型分布从实数轴 ℝ 上抽样，或者有时从高维（ℝ²、 ℝ³ 等等）抽取一个点。
Rand 针对大多数场景提供了 `f64` 和 `f32` 类型的实现。
目前 `f32` 类型的实现结果只是比 `f64` 类型的少了些精度。

* [`Exp`] 以固定衰减率 (exponential decay) 作为参数，模拟了衰退前的时间长度（寿命）——指数分布。
* [`Normal`] 需给定均值和标准差，模拟正态分布（高斯分布）抽样。
* [`LogNormal`] 对对数正态分布抽样，如果 `X` 服从对数正统分布，那么 `log(X)`
  服从正态分布。这种对正态分布变形，目的是避免负值而且希望有正坐标轴长尾。
* [`UnitCircle`] 和 [`UnitSphere`] 分布模拟从圆的边缘或者球面进行均匀抽样。
* [`Cauchy`] 分布 （Lorentz 分布）描述的是：从 `(x0, γ)` 点以均匀分布的角度散发一条射线，
  x 轴的截距则服从柯西分布。
* [`Beta`] 是两参数的概率分布，其随机值位于 0-1 区间内。
* [`Dirichlet`] 分布是对任意正参数的推广。

[Sequences]: ./guide-seq.html
[`Distribution`]: https://rust-random.github.io/rand/rand/distributions/trait.Distribution.html
[`distributions`]: https://rust-random.github.io/rand/rand/distributions/index.html
[`rand`]: https://rust-random.github.io/rand/rand/index.html
[`rand_distr`]: https://rust-random.github.io/rand/rand_distr/index.html
[`Rng::gen_range`]: https://rust-random.github.io/rand/rand/trait.Rng.html#method.gen_range
[`random`]: https://rust-random.github.io/rand/rand/fn.random.html
[`Rng::gen_bool`]: https://rust-random.github.io/rand/rand/trait.Rng.html#method.gen_bool
[`Rng::gen_ratio`]: https://rust-random.github.io/rand/rand/trait.Rng.html#method.gen_ratio
[`Rng::gen`]: https://rust-random.github.io/rand/rand/trait.Rng.html#method.gen
[`Rng`]: https://rust-random.github.io/rand/rand/trait.Rng.html
[`Standard`]: https://rust-random.github.io/rand/rand/distributions/struct.Standard.html
[`Uniform`]: https://rust-random.github.io/rand/rand/distributions/struct.Uniform.html
[`Uniform::sample_single`]: https://rust-random.github.io/rand/rand/distributions/struct.Uniform.html#method.sample_single
[`Alphanumeric`]: https://rust-random.github.io/rand/rand/distributions/struct.Alphanumeric.html
[`Open01`]: https://rust-random.github.io/rand/rand/distributions/struct.Open01.html
[`OpenClosed01`]: https://rust-random.github.io/rand/rand/distributions/struct.OpenClosed01.html
[`Bernoulli`]: https://rust-random.github.io/rand/rand/distributions/struct.Bernoulli.html
[`Binomial`]: https://rust-random.github.io/rand/rand/distributions/struct.Binomial.html
[`Exp`]: https://rust-random.github.io/rand/rand_distr/struct.Exp.html
[`Normal`]: https://rust-random.github.io/rand/rand_distr/struct.Normal.html
[`LogNormal`]: https://rust-random.github.io/rand/rand_distr/struct.LogNormal.html
[`LogNormal`]: https://rust-random.github.io/rand/rand_distr/struct.LogNormal.html
[`UnitCircle`]: https://rust-random.github.io/rand/rand_distr/struct.UnitCircle.html
[`UnitSphere`]: https://rust-random.github.io/rand/rand_distr/struct.UnitSphere.html
[`Cauchy`]: https://rust-random.github.io/rand/rand_distr/struct.Cauchy.html
[`Poisson`]: https://rust-random.github.io/rand/rand_distr/struct.Poisson.html
[`Beta`]: https://rust-random.github.io/rand/rand_distr/struct.Beta.html
[`Dirichlet`]: https://rust-random.github.io/rand/rand_distr/struct.Dirichlet.html
[`WeightedIndex`]: https://rust-random.github.io/rand/rand_distr/struct.WeightedIndex.html
[`alias_method::WeightedIndex`]: https://rust-random.github.io/rand/rand_distr/weighted_alias/struct.WeightedAliasIndex.html
