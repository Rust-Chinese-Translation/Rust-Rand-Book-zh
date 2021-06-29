# 生成器的类别

[`RngCore`] trait 是所有 <abbr title="random data sources">随机数据来源</abbr>
所必须实现的。但究竟什么是随机数据来源呢？

这一章涉及一些理论；可以参考 [支持的 RNGs](guide-rngs.md) 一章。

```rust
# extern crate rand;
# extern crate rand_pcg;
// prepare a non-deterministic random number generator:
let mut rng = rand::thread_rng();

// prepare a deterministic generator:
use rand::SeedableRng;
let mut rng = rand_pcg::Pcg32::seed_from_u64(123);
```

## 真随机数 TRNGs

一个真正的随机数生成器 (TRNG: **true** random number generator)
是通过观察某些自然过程，比如原子衰退或热噪声来生成随机数。
这些过程是真正随机的还是实际上确定的，
比如宇宙自身可以是一个模拟系统，但都不是这里我们所关心的。
对于我们来说，只要能与真正的随机性没有区别就足够了。

要注意，这些过程时常有倾向性，
所以必须采用某种消除倾向性 (debiasing) 的类型
来产生我们想要的无倾向性的随机数据。

## 伪随机数 PRNGs

CPU 当然应该确切地进行计算，然而事实证明，
它也能很好地模拟随机过程。
大多数伪随机数生成器 (PRNG: pseudo-random number generators) 
是确定性的，可以被定义为：

- 有某种初始化的状态 (state)
- 是一个从状态中计算随机值的函数
- 是一个进入下个状态的函数
- （可以是）从种子或者秘钥中获得初始状态的一个函数

这些伪随机数的确定性有时可以非常有用：
它让一场模拟行为、随机产生的艺术作品或游戏可以准确地重复；
产生的这种结果就是随机种子。
你可以在 [移植性](portability.md) 这一章阅读细节。
注意，仅仅是确定性还无法重现结果。

伪随机数另一个好处是速度很快：
有些算法需要对每个随机值只进行少量的 CPU 操作，
所以这比大多数真随机数生成器生成的速度快得多。

也要注意，PRNGs 有以下限制：

- 它们与随机种子一样不健壮：如果种子和算法已知或可推测，
  那么生成的序列中仅有一小部分结果是随机。
- 由于状态的大小通常是固定的，在生成器自身循环和重复的时候，
  仅能生成有限的随机数。
- 有些算法通过观测少量的值就能轻易被预测，
  而且这些算法之外，很多算法并不清楚会不会被破解。

## 加密级安全的 PRNGs

加密级安全的 PRNGs 
(CSPRNGs: Cryptographically secure pseudo-random number generators)
是 PRNGs 的子集，而且被认为是安全的。即

- 它们的状态足够多，以至于只尝试所有初始值的暴力破解方法，
  不太可能找到生成观测值的初始状态；
- 除了暴力破解方法之外，没有其他的算法可以足够好地预测下一个输出值。

安全的生成器 (CSPRNGs) 不仅需要安全的算法，
还需要安全且足够多的随机种子（通常是 256 位），
以及能抵挡住侧信道攻击（即防止攻击者读取内部状态）。

有些 CSPRNGs 还得满足第三种性质：抗回溯。
即使攻击者发现了当前内部状态的值，他也不可能计算出 PRNG 过去的输出值。
这暗含了，所有将来的输出值是缺乏抵抗力的。

## 基于硬件的 HRNGs

基于硬件的随机数生成器 (HRNG: **hardware** random number generator)
是理论上从某些 TRNG 到数字信息领域的延伸。
实际上， HRNGs 可能使用一个 PRNG 来消除 TRNG 的倾向性。
即使 HRNG 背后是 TRNG ，但 HRNG 不能保证是安全的。
TRNG 本身可能生成不了足够的熵（即很容易预测），
或者信号放大 (signal amplification) 和
去过程倾向性 (debias process) 可能有缺陷。

HRNGs 可能用来给 PRNGs 提供随机种子，
虽然这通常不是获取安全种子的唯一方式。
HRNGs 或许可以完全替代 PRNGs ，
虽然这通常不是一个满意的方法，
因为现在已经有非常快速和健壮的 PRNGs 软件，
而且软件实现比验证硬件要容易很多。

因为 PRNGs 需要随机种子必须是安全的，
所以 HRNGs 可以用来提供种子，或者甚至于替代 PRNGs 。
然而我们的目标通常只是产生不可预测的随机值，
还有一些可以替代 TRNGs 的方法。

## 信息熵 Entropy

正如上面提到的，一个 CSPRNG 要是安全的，其种子的值也必须是安全的。
熵可以在以下两个方面被使用：

- 衡量一堆数据中未知信息的的数量
- 当作一堆未知的数据

理想情况下，一个随机的布尔值或者抛掷一枚硬币有 1 bit 熵 (bit of entropy) ，
如果这个值有倾向性，熵的数量就会更少。
香农熵 (Shannon Entropy) 就试图来衡量这种情况。

比如一个 Unix 时间戳（从 1970 年以秒计算）包含了高精度和低精度的数据。
这通常是 32-bit 数字，但是熵的大小取决于一个假想攻击者 (hypothetical attacker)
可能以多少准确度猜出这个数字。
如果这个攻击者以接近分钟的准确度猜中数字，熵大约有 6 bits (2^6 = 64) ；
如果他猜中的准确度接近秒，则熵为 0 bit 。
[`JitterRng`] 生成器使用这种概念，不使用 HRNG 来清除熵；
它使用纳秒级精度的计时器，在对计时器的质量进行几次测试之后，
保守地假设每个时间戳只有一些熵。 

[`RngCore`]: https://rust-random.github.io/rand/rand_core/trait.RngCore.html
[`JitterRng`]: https://docs.rs/rand_jitter/*/rand_jitter/struct.JitterRng.html
