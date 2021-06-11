# 随机数生成器

## `Getrandom` crate

[`getrandom`] crate 依赖于平台，提供了低层次的 API 来生成随机数，
而且是 `rand` 、 `rand_core` 和一些加密学库的重要组成部分。
它可以不局限于用作成低层次的库。

有些情况，尤其针对 WASM 领域，使用者可能需要对这个 crate 进行一些配置。
所以请参考 [`getrandom`] 的文档。

## CPU 扰动

[`rand_jitter`] crate 实现了 <abbr title="CPU-jitter-based">基于 CPU 扰动</abbr> 
的信息熵，这在有高精度的 CPU 计时器的情况下，可用来作为提供熵来源的替代办法。

需要注意的是， CPU 抖动可能倾向于受到侧信道攻击 ([side-channel attacks]) ，
而且由于每个步骤对所获得的熵要进行保守地估计，
这个实现非常慢。

在 `rand` 早期版本，这个 crate 曾是一个直接依赖，
当其他的熵来源都失败的时候，自动被使用。
现在的版本，它不再是一个依赖了，甚至连可选依赖都算不上。

[side-channel attacks]:https://github.com/rust-random/rand/issues/699

## <abbr title="Deterministic generators">确定性的生成器</abbr>

以下 crates 实现了 <abbr title="pseudo-random">伪随机数</abbr> 生成器。
具体请查看 [生成器的分类](guide-rngs.md) 。

- [`rand_chacha`] 提供了 ChaCha 加密算法的的生成器
- [`rand_hc`] 提供了 HC-128 加密算法的的生成器
- [`rand_isaac`] 提供了 ISAAC 生成器
- [`rand_pcg`] 提供了一小部分的 PCG 生成器
- [`rand_xoshiro`] 提供了 SplitMix 和 Xoshiro 生成器
- [`rand_xorshift`] 提供了基础的 Xoshiro 生成器


[`rand_chacha`]: https://rust-random.github.io/rand/rand_chacha/index.html
[`rand_hc`]: https://rust-random.github.io/rand/rand_hc/index.html
[`rand_isaac`]: https://docs.rs/rand_isaac/
[`rand_pcg`]: https://rust-random.github.io/rand/rand_pcg/index.html
[`rand_xoshiro`]: https://docs.rs/rand_xoshiro/
[`rand_xorshift`]: https://docs.rs/rand_xorshift/
[`rand_jitter`]: https://docs.rs/rand_jitter/
[`getrandom`]: https://docs.rs/getrandom/
