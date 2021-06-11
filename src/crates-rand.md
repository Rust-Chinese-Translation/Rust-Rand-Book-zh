# Rand 与协作库

## `rand_core` 提供核心 API 

[`rand_core`] crate 定义给 RNGs 实现的核心的 traits 。
它作为单独的 crate 存在，具有两个目的：

- 提供定义和使用 RNGs 的最小 API 
- 给 RNGs 的 trait 实现提供工具

[`RngCore`], [`SeedableRng`], [`CryptoRng`] traits 和 [`Error`] 类型
全部在这个 crate 中被定义，然后重导出 (re-exported) 到 [`rand`] crate 。

## `rand` 主要的用户接口

[`rand`] crate 通过优化提供常用的随机数功能。这包括以下几个方面：

- [`rngs`] 模块提供一些方便的生成器
- [`distributions`] 模块主要用来对随机值抽样
- [`seq`] 模块主要用来从序列中抽样或者打乱序列 (shuffle sequences)
- [`Rng`] trait 提供一些方便生成随机值的方法
- [`random`] 函数 可通过单次调来生成随机数

### Feature flags

除了 [常见的 feature flags][common feature flags] ,
以下方面也是可以配置的：

- 使用 `small_rng` feature：开启 [`SmallRng`] 生成器 （Rand v0.7 之后默认关闭）
- 使用 `simd_support` feature：开启试验性的生成 SIMD 值功能
  （仅在 nightly Rust 中可用）\
  注意：关于 SIMD ，这个 flag 主要用于生成 SIMD 类型，而不进行优化。
  不管有没有开启这个 flag ，SIMD 操作都可能会在 Rand 库内部有所使用，比如
  在 ChaCha 生成器内部就地明确支持 SIMD 操作。

## 概率分布

[`rand`] crate 只实现了最常用的随机数分布：均匀分布和加权抽样。
对于其他的概率分布：

- [`rand_distr`] crate 提供了来自一系列分布的快速抽样功能，
  包括 正态分布（高斯分布）、二项分布、泊松分布、单位圆分布 等等。
- [`statrs`] crate 是 C# Math.NET 库的接口，
  不仅实现了与 [`rand_distr`] 类似的分布（有些多有些少），
  并且涵盖 PDF 和 CDF 函数，以及标准误、*beta*、*gamma* 和 *logistic* 等特殊函数，
  以及一些实用的东西。（ [`statrs`] 并不属于 Rand 库）

[common feature flags]: crates.md#feature-flags

[`rand_core`]: https://rust-random.github.io/rand/rand_core/index.html
[`rand`]: https://rust-random.github.io/rand/rand/index.html
[`rand_distr`]: https://rust-random.github.io/rand/rand_distr/index.html
[`statrs`]: https://github.com/boxtown/statrs

[`RngCore`]: https://rust-random.github.io/rand/rand_core/trait.RngCore.html
[`SeedableRng`]: https://rust-random.github.io/rand/rand_core/trait.SeedableRng.html
[`CryptoRng`]: https://rust-random.github.io/rand/rand_core/trait.CryptoRng.html
[`Error`]: https://rust-random.github.io/rand/rand_core/struct.Error.html

[`rngs`]: https://rust-random.github.io/rand/rand/rngs/index.html
[`distributions`]: https://rust-random.github.io/rand/rand/distributions/index.html
[`seq`]: https://rust-random.github.io/rand/rand/seq/index.html
[`Rng`]: https://rust-random.github.io/rand/rand/trait.Rng.html
[`random`]: https://rust-random.github.io/rand/rand/fn.random.html

[`SmallRng`]: https://rust-random.github.io/rand/rand/rngs/struct.SmallRng.html
