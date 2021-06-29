# 移植性

## 定义

给定输入，输出应该属于以下三类中的一类：

-   输出是非确定的，所以无法重现
-   输出是确定的，但不是可移植的
-   输出是确定的，而且是可移植的

一般而言，功能 (functionality) 是确定的、可移植的。
除非有些功能（比如 `getrandom`、 `ThreadRng`）显然不是确定的，
或者有些功能（比如 `StdRng`、 `SmallRng`）在文档中说明是不可移植的。

## crate 版本

Rand 尽量遵循 [版本规则](https://docs.npmjs.com/misc/semver) ，
即涉及不兼容的 API 时，遵循 `MAJOR.MINOR.PATCH` 版本号规则：

-   每次发布新的 *patch* （补丁）版本中，不应该有不兼容的 API 变动或添加主特性
-   在 1.0 版本前，每次发布新的 *minor* （次）版本中，可能有不兼容的 API 变动。 
-   在 1.0 版本之后，发布新的次版本中，不应该有不兼容的 API 变动。

此外，Rand 还必须考虑不兼容的值更改和移植性。给定输入：

-   对于非确定的条目，其实现可能在任何发行版中变动。
-   对于确定的、不可移植的条目，补丁版本应保持输出结果不变，次版本可以更改输出结果
    （包括 1.0 之后的次版本）。
-   对于可移植的条目，任何输出值的变动等同于非兼容性 API 变动。

### 测试

Rand 希望所有伪随机数算法在可能的时候能稳定地测试输出值：

-   PRNGs 应该和 vector 的引用进行比较。
    （[例子](https://github.com/rust-random/rngs/blob/master/rand_xoshiro/src/xoshiro256starstar.rs#L113)）
-   其他算法应该在 `value_stability` 测试或相似的测试代码中包含用于测试的 vector
    （[例子](https://github.com/rust-random/rand/blob/master/src/distributions/bernoulli.rs#L168)）

## 限制

### usize 的移植性

不巧，有一个 Rust 核心条目是不可移植的：`usize` （和 `isize`）。
比如一个空 `Vec` 在 32 位 和 64 位机器上大小不同。
对大多数使用场景，这没啥大问题，但涉及以移植的方式产生随机数时，这问题很大。

Rand 在任何可以遵循的时候，都遵循一条简单的规则：
如果需要移植性，就别直接对 `usize` 或 `isize` 类型的数据抽样。

所有涉及序列的代码中，要求 `usize` 边界值的时候， Rand 都会抽取 `u32` 的值，
除非序列的上界超过了 `u32::MAX` 。
注意，这其实在很多情况下提高了基准测试的性能。

### 浮点数的移植性

计算浮点数的结果取决于取整方式和实现的细节。
尤其是超越函数 (transcendental functions) 的计算结果在每个平台上都不一样。
所以， `rand_distr` crate 里的分布对 `f32` 和 `f64` 类型不总是可移植的，
但 Rand 尽量让浮点数具有移植性。
