# 随机值

我们已经有办法生成随机数据，但如何把数据转化成我们想要的类型的值呢？

这是一个难题：我们既需要知道 **范围** ，又需要知道这个值的 **分布** 。
对于分布，将在 [下一章](guide-dist.md) 介绍。

## `Rng` trait

方便起见，所有的 RNG 自动实现了 [`Rng`] trait ，从而方便地生成随机值。
以下是方便生成均匀分布随机值的一些函数（方法）：

-   [`Rng::gen`] 从类型合适的范围内生成无偏均匀分布的随机值。
    对于整数，通常在整个范围内产生值（比如从 `0u32` 到 `std::u32::MAX` 生成 `u32` 类型的值）；
    对于浮点数，从 0-1 内产生值；也支持一些其他类型，比如数组和元组。
    
    这个方法能很方便地生成 [`Standard`] 分布（译者注：这不是正态分布/标准正态分布，
    而是 Rand 库自定义的一种拓展形式上的“均匀分布”） ，这在
    [下一章](guide-dist.html#uniform-distributions) 会介绍。
-   [`Rng::gen_range`] 在给定的范围内生成无偏随机值。
-   [`Rng::fill`] 和 [`Rng::try_fill`] 是经过优化的函数，用来给字节或者整数切片填充随机值。

以下方法能方便生成非均匀分布的 bool 值：

-   [`Rng::gen_bool`] 以给定概率生成一个 bool 值 
-   [`Rng::gen_ratio`] 以分数形式的概率生成一个 bool 值

最后，这个函数能从任意分布中抽样产生随机值：

-   [`Rng::sample`] 直接从某些 [分布](guide-dist.md) 中抽样

例子：

```rust
# extern crate rand;
use rand::Rng;
let mut rng = rand::thread_rng();

// an unbiased integer over the entire range:
let i: i32 = rng.gen();

// a uniformly distributed value between 0 and 1:
let x: f64 = rng.gen();

// simulate rolling a die:
let roll = rng.gen_range(1..7);
```

此外， [`random`] 函数基于 [`thread_rng`] 来调用 [`Rng::gen`] ，是产生随机值的捷径。

```rust
if rand::random() {
    println!("we got lucky!");
}
```

[`Rng`]: https://rust-random.github.io/rand/rand/trait.Rng.html
[`Rng::gen`]: https://rust-random.github.io/rand/rand/trait.Rng.html#method.gen
[`Rng::gen_range`]: https://rust-random.github.io/rand/rand/trait.Rng.html#method.gen_range
[`Rng::sample`]: https://rust-random.github.io/rand/rand/trait.Rng.html#method.sample
[`Rng::gen_bool`]: https://rust-random.github.io/rand/rand/trait.Rng.html#method.gen_bool
[`Rng::gen_ratio`]: https://rust-random.github.io/rand/rand/trait.Rng.html#method.gen_ratio
[`Rng::fill`]: https://rust-random.github.io/rand/rand/trait.Rng.html#method.fill
[`Rng::try_fill`]: https://rust-random.github.io/rand/rand/trait.Rng.html#method.try_fill
[`random`]: https://rust-random.github.io/rand/rand/fn.random.htm
[`thread_rng`]: https://rust-random.github.io/rand/rand/fn.thread_rng.html
[`Standard`]: https://rust-random.github.io/rand/rand/distributions/struct.Standard.html
