# 处理 Error 

Rand 库里面的错误处理折中考虑了简便性和必要性。
大多数 RNGs 和抽样函数不会产生 erros 。
如果处理错误，则可能导致不容忽视的开销，至少会导致代码更复杂、使用更不人性化，
以及可能对性能造成影响。
但是，内部的 RNGs 还是有可能运行失败，所以处理这种情况的错误是很重要的。

Rand 里大多数方法不应该返回 `Result` 类型，但以下是一些重要的例外：

-   [`Rng::try_fill`]
-   [`RngCore::try_fill_bytes`]
-   [`SeedableRng::from_rng`]

大多数消耗随机值的函数也不会做任何错误处理，
而且在底层调用 [`RngCore`] trait 的 “永不运行失败” 的方法。
因为大部分 RNGs 不会运行失败，所以这种机制不会带来太大问题，
但是少数一些生成器却可能运行失败：

-   [`OsRng`] 对接操作系统的生成器的接口；在极少数的情况下，它会因为没准备好或者不可用而 fail 。
-   [`JitterRng`] 基于计时器扰动 (timer jitter) 的生成器；如果计时器缺乏精度或者太容易预测，它会 fail 。
-   [`EntropyRng`] 是对上面两个的抽象，所以它们 fail 的时候，这个最终会 fail。（译者注，这个已经 [deprecated](https://github.com/rust-random/rand/pull/765) 了）
-   [`thread_rng`] 通过 [`EntropyRng`] 来设置种子，当每个线程第一次使用的时候可能会 fail，
    第一次使用之后绝不会 fail 。
-   [`ReadRng`] 尝试从来源中读取数据，当流结束或报错时，它会 fail 。
    （译者注，这个也 deprecated 了）

[`Rng::try_fill`]: https://rust-random.github.io/rand/rand/trait.Rng.html#method.try_fill
[`RngCore::try_fill_bytes`]: https://rust-random.github.io/rand/rand_core/trait.RngCore.html#tymethod.try_fill_bytes
[`SeedableRng::from_rng`]: https://rust-random.github.io/rand/rand_core/trait.SeedableRng.html#method.from_rng
[`RngCore`]: https://rust-random.github.io/rand/rand_core/trait.RngCore.html
[`thread_rng`]: https://rust-random.github.io/rand/rand/fn.thread_rng.html
[`RngCore`]: https://rust-random.github.io/rand/rand_core/trait.RngCore.html
[`thread_rng`]: https://rust-random.github.io/rand/rand/fn.thread_rng.html
[`OsRng`]: https://rust-random.github.io/rand/rand/rngs/struct.OsRng.html
[`JitterRng`]: https://docs.rs/rand_jitter/*/rand_jitter/struct.JitterRng.html
[`EntropyRng`]: https://rust-random.github.io/rand/rand/rngs/struct.EntropyRng.html
[`ReadRng`]: https://rust-random.github.io/rand/rand/rngs/adapter/struct.ReadRng.html
