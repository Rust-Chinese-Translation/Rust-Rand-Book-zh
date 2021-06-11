# 支持的 RNGs

有很多种 RNGs ，它们具有各自的优缺点。
Rand 库在 [`rngs`] 模块 里提供了一些方便的生成器。
通常你可以使用 [`thread_rng`] 函数，它会在本地线程内存中自动初始化一个 RNG ，
然后返回它的引用；它很快，质量不错，而且（据我们所知）是加密级安全的。

这页文档的目录：

1. [The generators](#the-generators)
1. [Performance and size](#performance)
1. [Quality and cycle length](#quality)
1. [Security](#security)
1. [Extra features](#extra-features)
1. [Further reading](#further-reading)

# 介绍生成器

## 基础的 PRNGs

“标准的”非加密级别的 PRNGs 
的目标常常是在易用、质量、内存占用和性能之间找到良好的平衡点。
非加密级别的生成器时间上先于加密级别的生成器，
在某些方面已被加密级的淘汰，但是非加密级别的生成器有以下一些优势：
更小的状态，快速初始化、使用简单、在嵌入式 CPUs 上更小的内存占用。
然而不是所有非加密级的 PRNGs 提供这些优点，
比如 Mersenne Twister 算法虽容易预测，但有很大的状态。

这些算法对于 Monte Carlo 模拟是很重要的，
而且也适合像随机化算法、随机化游戏这样场景，没有预测问题。
注意，赌博游戏可能具有预测方面问题，从而推荐加密级的 PRNGs 。

Rand 库提供的非加密级 PRNGs 可以用下面的表格总结。
你可能需要参考 [pcg-random] 和 [xoshiro] 。

| 名称                   | 全名                   | 性能    | 内存     | 质量  | 范围                        | 特点       |
| --                     | --                     | --      | --       | --    | --                          | --         |
| [`SmallRng`]           | 无                     | 7 GB/s  | 16 bytes | ★★★☆☆ | ≥ `u32` * 2<sup>64</sup>    | 无移植性   |
| [`Pcg32`]              | PCG XSH RR 64/32 (LCG) | 3 GB/s  | 16 bytes | ★★★☆☆ | `u32` * 2<sup>64</sup>      | —          |
| [`Pcg64`]              | PCG XSL 128/64 (LCG)   | 4 GB/s  | 32 bytes | ★★★☆☆ | `u64` * 2<sup>128</sup>     | —          |
| [`Pcg64Mcg`]           | PCG XSL 128/64 (MCG)   | 7 GB/s  | 16 bytes | ★★★☆☆ | `u64` * 2<sup>126</sup>     | —          |
| [`XorShiftRng`]        | Xorshift 32/128        | 5 GB/s  | 16 bytes | ★☆☆☆☆ | `u32` * 2<sup>128</sup> - 1 | —          |
| [`Xoshiro256PlusPlus`] | Xoshiro256++           | 7 GB/s  | 32 bytes | ★★★☆☆ | `u64` * 2<sup>256</sup> - 1 | jump-ahead |
| [`Xoshiro256Plus`]     | Xoshiro256+            | 8 GB/s  | 32 bytes | ★★☆☆☆ | `u64` * 2<sup>256</sup> - 1 | jump-ahead |
| [`SplitMix64`]         | splitmix64             | 8 GB/s  | 8 bytes  | ★☆☆☆☆ | `u64` * 2<sup>64</sup>      | —          |
| [`StepRng`]            | counter                | 51 GB/s | 16 bytes | ☆☆☆☆☆ | `u64` * 2<sup>64</sup>      | —          |

这里性能的衡量方式是在 3.4GHz Haswell CPU 上粗略地进行 `u64` 类型的输出；
注意性能在不同的应用场景下会明显不同；
一般来说，加密级的 RNGs 会更好地处理字节序列输出。

质量评级是基于理论支持和可观测到的缺陷两个角度，粗略地分成：

-   ★☆☆☆☆ = 适合简单的应用场景，但是有具有的缺陷
-   ★★☆☆☆ = 在大多数测试中性能较好，有一些问题
-   ★★★☆☆ = 性能和理论支持都很好，没什么大问题
-   ★★★★★ = 加密级的质量

## 加密级安全的 RNGs (CSPRNGs)

CSPRNGs 比基础的 PRNGs 有更高的要求。主要是对安全的考虑。
性能和易用也很重要，但是 CSPRNGs 一般更复杂一些，比常规的 PRNGs 更慢。
质量不再是关心的重点，因为 CSPRNGs 生成的随机数基本上与真的随机值别无二致，
这正是它所要求的。毕竟任何的倾向性或相关性都会让输出结果更容易被预测。

CSPRNGs 和加密密码 (cryptographic ciphers) 具有密切的联系。
任何块密码 (block cipher) 都能转化为对一个计数器 (counter) 加密的 CSPRNG 。
流密码 (stream cipher) 基本上就是一个 CSPRNG 和一组操作（通常是 XOR）。
这意味着，我们能很容易地把任何流密码用作一个 CSPRNG 。

Rand 库提供了以下 CSPRNGs 。可以不再要求任何安全性（因为它们已经是安全的）。
这张表去掉了 “质量” 一列，因为 CSPRNGs 可能不会有可观测到的缺陷。

| 名称            | 全名     | 性能     | 初始化 | 内存       | 安全性（可预测性） | 前向保密性 |
| --              | --       | --       | --     | --         | --                 | --         |
| [`StdRng`]      | 无       | 1.5 GB/s | 快     | 136 bytes  | 被广泛信赖         | 无         |
| [`ChaCha20Rng`] | ChaCha20 | 1.8 GB/s | 快     | 136 bytes  | [有严格的分析]     | 无         |
| [`ChaCha8Rng`]  | ChaCha8  | 2.2 GB/s | 快     | 136 bytes  | 安全性很小         | 无         |
| [`Hc128Rng`]    | HC-128   | 2.1 GB/s | 慢     | 4176 bytes | [被 eSTREAM 推荐]  | 无         |
| [`IsaacRng`]    | ISAAC    | 1.1 GB/s | 慢     | 2072 bytes | [未知]             | 未知       |
| [`Isaac64Rng`]  | ISAAC-64 | 2.2 GB/s | 慢     | 4136 bytes | 未知               | 未知       |

[有严格的分析]: https://tools.ietf.org/html/rfc7539#section-1
[被 eSTREAM 推荐]: http://www.ecrypt.eu.org/stream/
[未知]: https://burtleburtle.net/bob/rand/isaacafa.html

应该十分注意，ISAAC 生成器仅出于历史原因使用：
它们从一开始就和 Rust 语言一起被使用，
它们可以良好地生成随机数，也没有被攻击成功过，
但是并未得到加密学专家的关注。

# 评价生成器

## 性能

First it has to be said most PRNGs are very fast, and will rarely be a
performance bottleneck.

Performance of basic PRNGs is a bit of a subtle thing. It depends a lot on
the CPU architecture (32 vs. 64 bits), inlining, and also on the number of
available registers. This often causes the performance to be affected by
surrounding code due to inlining and other usage of registers.

When choosing a PRNG for performance it is important to benchmark your own
application due to interactions between PRNGs and surrounding code and
dependence on the CPU architecture as well as the impact of the size of
data requested. Because of all this, we do not include performance numbers
here but merely a qualitative rating.

CSPRNGs are a little different in that they typically generate a block of
output in a cache, and pull outputs from the cache. This allows them to have
good amortised performance, and reduces or completely removes the influence
of surrounding code on the CSPRNG performance.

### Worst-case performance
Simple PRNGs typically produce each random value on demand. In contrast, CSPRNGs
usually produce a whole block at once, then read from this cache until it is
exhausted, giving them much less consistent performance when drawing small
quantities of random data.

### Memory usage

Simple PRNGs often use very little memory, commonly only a few words, where
a *word* is usually either `u32` or `u64`. This is not true for all
non-cryptographic PRNGs however, for example the historically popular
Mersenne Twister MT19937 algorithm requires 2.5 kB of state.

CSPRNGs typically require more memory; since the seed size is recommended
to be at least 192 bits and some more may be required for the algorithm,
256 bits would be approximately the minimum secure size. In practice,
CSPRNGs tend to use quite a bit more, [`ChaChaRng`] is relatively small with
136 bytes of state.

### Initialization time

The time required to initialize new generators varies significantly. Many
simple PRNGs and even some cryptographic ones (including [`ChaChaRng`])
only need to copy the seed value and some constants into their state, and
thus can be constructed very quickly. In contrast, CSPRNGs with large state
require an expensive key-expansion.

## Quality

Many basic PRNGs are not much more than a couple of bitwise and arithmetic
operations. Their simplicity gives good performance, but also means there
are small regularities hidden in the generated random number stream.

How much do those hidden regularities matter? That is hard to say, and
depends on how the RNG gets used. If there happen to be correlations between
the random numbers and the algorithm they are used in, the results can be
wrong or misleading.

A random number generator can be considered good if it gives the correct
results in as many applications as possible. The quality of PRNG
algorithms can be evaluated to some extent analytically, to determine the
cycle length and to rule out some correlations. Then there are empirical
test suites designed to test how well a PRNG performs on a wide range of
possible uses, the latest and most complete of which are [TestU01] and
[PractRand].

CSPRNGs tend to be more complex, and have an explicit requirement to be
unpredictable. This implies there must be no obvious correlations between
output values.

### Quality stars:
PRNGs with 3 stars or more should be good enough for most non-crypto
applications. 1 or 2 stars may be good enough for typical apps and games, but do
not work well with all algorithms.

### Period

The *period* or *cycle length* of a PRNG is the number of values that can be
generated after which it starts repeating the same random number stream.
Many PRNGs have a fixed-size period, while for others ("chaotic RNGs") the
cycle length may depend on the seed and short cycles may exist.

Note that a long period does not imply high quality (e.g. a counter through
`u128` values provides a decently long period). Conversely, a short period may
be a problem, especially when multiple RNGs are used simultaneously.
In general, we recommend a period of at least 2<sup>128</sup>.
(Alternatively, a PRNG with shorter period of at least 2<sup>64</sup> and
support for multiple streams may be sufficient. Note however that in the case
of PCG, its streams are closely correlated.)

*Avoid reusing values!*
On today's hardware, a fast RNG with a cycle length of *only*
2<sup>64</sup> can be used sequentially for centuries before cycling. However,
when multiple RNGs are used in parallel (each with a unique seed), there is a
significant chance of overlap between the sequences generated.
For a generator with a *large* period `P`, `n` independent generators, and
a sequence of length `L` generated by each generator, the chance of any overlap
between sequences can be approximated by `Ln² / P` when `nL / P` is close to
zero. For more on this topic, please see these
[remarks by the Xoshiro authors](http://prng.di.unimi.it/#remarks).

*Collisions and the birthday paradox!*
For a generator with outputs of equal size to its state, it is recommended not
to use more than `√P` outputs. A generalisation for `kw`-bit state and `w`-bit
generators is to ensure `kL² < P`. This requirement stems from the
*generalised birthday problem*, asking how many unbiased samples from a set of
size `d = 2^w` can be taken before the probability of a repeat is at least half.
Note that for `kL² > P` a generator with `kw`-dimensional equidistribution
*cannot* generate the expected number of repeated samples, however generators
without this property are *also* not guaranteed to generate the expected number
of repeats.

## Security

### Predictability

From the context of any PRNG, one can ask the question *given some previous
output from the PRNG, is it possible to predict the next output value?*
This is an important property in any situation where there might be an
adversary.

Regular PRNGs tend to be predictable, although with varying difficulty. In
some cases prediction is trivial, for example plain Xorshift outputs part of
its state without mutation, and prediction is as simple as seeding a new
Xorshift generator from four `u32` outputs. Other generators, like
[PCG](http://www.pcg-random.org/predictability.html) and truncated Xorshift*
are harder to predict, but not outside the realm of common mathematics and a
desktop PC.

The basic security that CSPRNGs must provide is the infeasibility to predict
output. This requirement is formalized as the [next-bit test]; this is
roughly stated as: given the first *k* bits of a random sequence, the
sequence satisfies the next-bit test if there is no algorithm able to
predict the next bit using reasonable computing power.

A further security that *some* CSPRNGs provide is forward secrecy:
in the event that the CSPRNGs state is revealed at some point, it must be
infeasible to reconstruct previous states or output. Note that many CSPRNGs
*do not* have forward secrecy in their usual formulations.

Verifying security claims of an algorithm is a *hard problem*, and we are not
able to provide any guarantees of the security of algorithms used or recommended
by this project. We refer you to the [NIST] institute and [ECRYPT] network
for recommendations.

### State and seeding

It is worth noting that a CSPRNG's security relies absolutely on being
seeded with a secure random key. Should the key be known or guessable, all
output of the CSPRNG is easy to guess. This implies that the seed should
come from a trusted source; usually either the OS or another CSPRNG. Our
seeding helper trait, [`FromEntropy`], and the source it uses
([`EntropyRng`]), should be secure. Additionally, [`ThreadRng`] is a CSPRNG,
thus it is acceptable to seed from this (although for security applications
fresh/external entropy should be preferred).

Further, it should be obvious that the internal state of a CSPRNG must be
kept secret. With that in mind, our implementations do not provide direct
access to most of their internal state, and `Debug` implementations do not
print any internal state. This does not fully protect CSPRNG state; code
within the same process may read this memory (and we allow cloning and
serialisation of CSPRNGs for convenience). Further, a running process may be
forked by the operating system, which may leave both processes with a copy
of the same generator.

### Not a crypto library

It should be emphasised that this is not a cryptography library; although
Rand does take some measures to provide secure random numbers, it does not
necessarily take all recommended measures. Further, cryptographic processes
such as encryption and authentication are complex and must be implemented
very carefully to avoid flaws and resist known attacks. It is therefore
recommended to use specialized libraries where possible, for example
[openssl], [ring] and the [RustCrypto libraries].


## Extra features

Some PRNGs may provide extra features, like:

- Support for multiple streams, which can help with parallel tasks.
- The ability to jump or seek around in the random number stream;
with a large period this can be used as an alternative to streams.


## Further reading

There is quite a lot that can be said about PRNGs. The [PCG paper] is very
approachable and explains more concepts.

Another good paper about RNG quality is
["Good random number generators are (not so) easy to find"](
https://web.archive.org/web/20160801142711/http://random.mat.sbg.ac.at/results/peter/A19final.pdf)
by P. Hellekalek.


[`rngs`]: https://rust-random.github.io/rand/rand/rngs/index.html
[`SmallRng`]: https://rust-random.github.io/rand/rand/rngs/struct.SmallRng.html
[`StdRng`]: https://rust-random.github.io/rand/rand/rngs/struct.StdRng.html
[`StepRng`]: https://rust-random.github.io/rand/rand/rngs/mock/struct.StepRng.html
[`thread_rng`]: https://rust-random.github.io/rand/rand/fn.thread_rng.html
[basic PRNGs]: #basic-pseudo-random-number-generators-prngs
[CSPRNGs]: #cryptographically-secure-pseudo-random-number-generators-csprngs
[`Pcg32`]: https://rust-random.github.io/rand/rand_pcg/type.Pcg32.html
[`Pcg64`]: https://rust-random.github.io/rand/rand_pcg/type.Pcg64.html
[`Pcg64Mcg`]: https://rust-random.github.io/rand/rand_pcg/type.Pcg64Mcg.html
[`XorShiftRng`]: https://docs.rs/rand_xorshift/latest/rand_xorshift/struct.XorShiftRng.html
[`Xoshiro256PlusPlus`]: https://docs.rs/rand_xoshiro/latest/rand_xoshiro/struct.Xoshiro256PlusPlus.html
[`Xoshiro256Plus`]: https://docs.rs/rand_xoshiro/latest/rand_xoshiro/struct.Xoshiro256Plus.html
[`SplitMix64`]: https://docs.rs/rand_xoshiro/latest/rand_xoshiro/struct.SplitMix64.html
[`ChaChaRng`]: https://rust-random.github.io/rand/rand_chacha/struct.ChaChaRng.html
[`ChaCha20Rng`]: https://rust-random.github.io/rand/rand_chacha/struct.ChaCha20Rng.html
[`ChaCha8Rng`]: https://rust-random.github.io/rand/rand_chacha/struct.ChaCha8Rng.html
[`Hc128Rng`]: https://rust-random.github.io/rand/rand_hc/struct.Hc128Rng.html
[`IsaacRng`]: https://docs.rs/rand_isaac/latest/rand_isaac/isaac/struct.IsaacRng.html
[`Isaac64Rng`]: https://docs.rs/rand_isaac/latest/rand_isaac/isaac64/struct.Isaac64Rng.html
[`ThreadRng`]: https://rust-random.github.io/rand/rand/rngs/struct.ThreadRng.html
[`FromEntropy`]: https://rust-random.github.io/rand/rand/trait.FromEntropy.html
[`EntropyRng`]: https://rust-random.github.io/rand/rand/rngs/struct.EntropyRng.html
[TestU01]: http://simul.iro.umontreal.ca/testu01/tu01.html
[PractRand]: http://pracrand.sourceforge.net/
[pcg-random]: http://www.pcg-random.org/
[xoshiro]: http://xoshiro.di.unimi.it/
[PCG paper]: http://www.pcg-random.org/pdf/hmc-cs-2014-0905.pdf
[openssl]: https://crates.io/crates/openssl
[ring]: https://crates.io/crates/ring
[RustCrypto libraries]: https://github.com/RustCrypto
[next-bit test]: https://en.wikipedia.org/wiki/Next-bit_test
[NIST]: https://www.nist.gov/
[ECRYPT]: http://www.ecrypt.eu.org/
