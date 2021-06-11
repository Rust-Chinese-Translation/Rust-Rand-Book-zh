# crates 和 features

## crates 介绍

Rand 库由一系列的 crates 组成。
[`rand`] crate 提供了主要的用户接口；
如果需要额外的随机分布，需要添加使用 [`rand_distr`] 或 [`statrs`] crate 。

Rand 库包含了好几个部分：
1. [`getrandom`] 接口提供依赖于平台的随机数代码；
2. [`rand_core`] 提供的 API 必须基于已实现的随机数生成器；
3. [`rand_chacha`] 和 [`rand_xoshiro`] 提供伪随机数生成器。

```
getrandom ┐
          └ rand_core ┐
                      ├ rand_chacha ┐
                      ├ rand_hc     ┤
                      ├ rand_pcg    ┤
                      └─────────────┴ rand ┐
                                           ├ rand_distr
                                           └ statrs
```

## feature flags

Rand 库可以通过 feature flags 进行配置。
查看各自 crate 的 README 以了解详细信息。

无标准库依赖 (no-std) 来使用大多数 Rand crates 的做法：
禁用默认的 features `rand = { version = "0.7", default-features = false  }` 。
这关闭了以下 flags ：
- `std` 标准库依赖
- `alloc` 分配器 (allocator) 功能（由 `std` 带来的，
  如果在 `no_std` feature 下开启这个功能，则 Rand 要求 Rustc 版本至少为 1.36 ）

有些 Rand crates 可以与以下第三方 crates 一起构建：
- [`log`] crate 的 `log` （提供一些日志消息）
- [`serde`] crate 的 `serde1` （提供序列化功能 serialization ）

注意，加密式 RNGs **不支持** 序列化，因为这可能导致安全风险。
如果你需要对加密式 RNGs 的状态存储功能，ChaCha 生成器支持 
[获取和设置流位置][getting and setting the stream position] ，
从而与随机种子一起使用，就能重现生成器的状态。

[getting and setting the stream position]:https://rust-random.github.io/rand/rand_chacha/struct.ChaCha20Rng.html#method.get_word_pos


## 支持 WASM 

几乎所有的 Rand crates 都开箱即用地支持 WASM 。
只有 `rand_core` crate 可能需要对支持 WASM 开启 [features][WASM-features] 。
也就是说，如果你使用基于 `rand_core` 的其他 crate （比如大多数 Rand crates ），
那么你可能要明确开启 `getrandom` features 来支持 WASM 。

[WASM-features]:https://docs.rs/getrandom

[`rand_core`]: https://rust-random.github.io/rand/rand_core/index.html
[`rand`]: https://rust-random.github.io/rand/rand/index.html
[`rand_distr`]: https://rust-random.github.io/rand/rand_distr/index.html
[`statrs`]: https://github.com/boxtown/statrs
[`getrandom`]: https://docs.rs/getrandom/
[`rand_chacha`]: https://rust-random.github.io/rand/rand_chacha/index.html
[`rand_xoshiro`]: https://docs.rs/rand_xoshiro/
[`log`]: https://docs.rs/log/
[`serde`]: https://serde.rs/
