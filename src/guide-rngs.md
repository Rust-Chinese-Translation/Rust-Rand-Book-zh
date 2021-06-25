# 支持的 RNGs

有很多种 RNGs ，它们具有各自的优缺点。
Rand 库在 [`rngs`] 模块 里提供了一些方便的生成器。
通常你可以使用 [`thread_rng`] 函数，它会在本地线程内存中自动初始化一个 RNG ，
然后返回它的引用；它很快，质量不错，而且（据我们所知）是加密级安全的。

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
| [`ChaCha8Rng`]  | ChaCha8  | 2.2 GB/s | 快     | 136 bytes  | 安全的边界很小     | 无         |
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

首先，不得不说 Rand 的大多数 PRNGs 非常快，几乎不会出现性能瓶颈。

基础 PRNGs 的性能有些微妙的地方。
它们的性能大多取决于 CPU 架构（32 或 64 位）、内联能力 (inlining) 和 寄存器 (register) 的数量。
性能常常因为内联和其他寄存器的使用情况，而受到上下文代码的影响。

出于性能考虑来选择一个 PRNG 的时候，对你的应用进行基准测试 (benchmark) 是很重要的，
因为 PRNGs 和上下文代码的交互、 CPU 架构以及所需数据的大小都会影响性能。
正因为如此，这里不会用数字来量化性能，而仅仅只是以定性评级的方式来衡量性能。

CSPRNGs 略微特别，因为它们通常在缓存里生成一块输出，然后从缓存中提取输出。
这种方式让 CSPRNGs 很好地摊销掉性能，从而减少或者完全消除了上下文代码对 CSPRNGs 的性能影响。

### 最坏情况下

简单的 PRNGs 一般按需生成单个随机值。
而 CSPRNGs 通常一次生成一整块随机值，然后在这块值消耗完之前，从缓存中读取这些值。
这使得在生成少量随机数据的时候， CSPRNGs 与 PRNGs 的性能差异较大。

### 内存使用

简单的 PRNGs 一般使用很少的内存，只需要几个字 (words) ，而一个字一般要么是 `u32` 要么是 `u64` 。
然而，也不是说所有非加密级别的 PRNGs 均如此。
比如历史上一度流行的 Mersenne Twister MT19937 算法就需要 2.5 kB (20480 bits) 大小的状态。

CSPRNGs 一般需要更多的内存，因为随机种子的大小建议至少为 192 bits ，
有些算法需要更多 bit 的种子，而 256 bits 是基本满足安全性的种子大小。
实际应用中， CSPRNGs 倾向于使用更多 bits 的种子，
[`ChaChaRng`] 使用 136 bytes (1088 bits) 大小的状态。

### 初始化时间

初始化新生成器的时间差异很明显。
很多简单的 PRNGs 、甚至某些加密级 PRNGs （包括 [`ChaChaRng`] ）都只需要
把种子的值和一些常量复制进状态里面，因此它们能很快地构造一个生成器。
而具有很大状态的 CSPRNGs 正相反，需要长时间的键展开 (key-expansion) 。

## 随机数的质量

许多基础的 PRNGs 只不过是一些位运算和算术运算。
它们的简单性提供了良好的性能，但也意味着在生成的随机数流中隐藏了一些小的规则。

这些隐藏的规则会造成多大影响呢？
很难讲。那取决于你如何 RNGs 。
随机数和这个随机数所使用的算法碰巧具有相关性的话，
RNGs 的结果可能就不正确，或者是误导的。

当某个 RNG 在许多应用尽可能给出正确的结果时，就被认为是一个好的 RNG 。
PRNG 算法的质量可以在一定程度上用解析 (analytically) 的方法来评估，
以确定周期长度并排除某些相关性。
也有一些基于经验的测试套件，用以测试 RNGs 在各种使用环境中的表现，
其中最新、最完整的是 [TestU01] 和 [PractRand] 。

CSPRNGs 往往复杂很多，它们明确要求随机值不可预测。
这表明输出值必须完全不相关。

### 质量得分

对于大多数非加密级应用的 PRNGs 可以评 3 星或更多。

对于一般的应用和游戏，不能与所有算法一起良好工作的 PRNGs 可以评 1 星或 2 星。

### 周期长度

一个 PRNG 的周期 (period) 或周期长度 (cycle length) 指：
在这个 PRNG 开始重复同一个随机数流之后，可被生成的随机值的数量。
许多 PRNGs 具有固定大小的周期，而诸如 chaotic RNGs 的周期长度可能取决于随机种子，
而且其周期可能很短。

要注意，长周期不意味着高质量，比如一个 `u128` 值的计数就提供了一个合适长度的周期。
但与之相反的是，短周期可能就会有带来问题，尤其在多个 RNGs 同时被使用的时候。
一般而言，建议一个周期至少有 2<sup>128</sup> 那么长。
或者说，一个短周期的 PRNG 应至少有 2<sup>64</sup> 的长度，而且支持多个流 (streams) 。
在 PCG 中，要注意它的流是密切相关的。

**避免重复使用值！**
在现代硬件上，一个仅有 2<sup>64</sup> 周期长度的 RNG 可在循环（重复）之前快速生成很长一列随机数。
而当多个 RNGs 并行以各自单独的种子被使用的时候，生成的序列很有可能重复 (overlap) 。
假设一个巨大周期 `P` 的 RNG，以及 `n` 个 RNGs ，这 n 个 RNGs 里每个都具有周期 `L` ，
那么当 `nL / P` 接近于 0 时，它们所产生的序列出现重复的的概率接近 `Ln² / P` 。
对这个问题的深入探讨，请参考：
[Xoshiro 作者们的讨论](http://prng.di.unimi.it/#remarks) 。

**碰撞问题 (collisions) 和生日悖论 (birthday paradox) ！**
如果生成器的输出值与其状态有相同的大小，
那么建议不要让生成器产生多于 `√P` 数量的随机值（ `P` 是 RNG 的周期长度）。
一般来说，具有 `kw`-bit 大小的状态和 `w`-bit 的 RNG 之间应确保 `kL² < P` 这样的关系。
这个要求源自于广义的生日问题 (generalised birthday problem) ，
这个问题求从一组大小为 `d = 2^w` 的总体中应抽取多少样本才能保证出现同一天生日的概率至少为 1/2 。
要注意的是，对于 `kL² > P` 的情况，虽然具有 `kw` 维度的均匀分布的 RNG 不可能产生预计重复样本，
但是不是多维均匀分布的 RNGs 也不能包装产生预计重复的数字。


## 安全性

### 可预测性

无论哪种 PRNG ，它们都面临一个问题：
**假如已知这个 PRNG 以前的一些输出值，还可能预测接下来的输出值吗？**
在任何可能出现敌手的情况下，（不）可预测性是一个很重要的性质。

常规的 PRNGs 往往可以被预测，虽然预测准确的话可能要经历重重困难。
有些情况下，预测可以轻而易举，比如单纯的 (plain) Xorshift 不加更改地输出其状态的时候，
只需四个 `u32` 输出值就能像设置了随机种子一样把 Xorshift 完全预测出来。
预测像 [PCG](http://www.pcg-random.org/predictability.html) 或者截断式 (truncated) Xorshift 
这样的 RNGs 就困难得多，但是利用通常的数学知识和一台台式电脑，还是可以做到预测出它们的。

CSPRNGs 必须提供的最基础的安全保证是：预测输出值是完全不可能的。
这个要求以 [next-bit test] 作为检验的标准，简单来说：
给定一串随机序列的前 k bits ，如果没有算法能在合理的计算能力之下预测下一个 bit ，
那么就称这个序列通过了 next-bit 测试。

某些 CSPRNGs 提供了更高的安全要求：向前保密 (forward secrecy) ,
某时刻 CSPRNGs 的状态被暴露的情况下，依然不可能构造之前的状态或者输出值。
注意，许多 CSPRNGs 通常在设计的时候并不考虑向前保密性。

验证某个算法宣称的安全性是一个困难的，Rand 无法保证提供或推荐的算法的安全性。
建议你参考 [NIST] 机构和 [ECRYPT] 。

### 状态和种子

值得注意的是，CSPRNGs 的安全性完全取决于提供给它的安全秘钥种子。
如果秘钥已知或者可以被推测，那么这个 CSPRNG 所有的输出值都容易被推测出来。
这意味着种子应该来源于可信赖的地方，通常要么是操作系统，要么是另一个的 CSPRNG 。

Rand 提供的种子辅助 trait [`FromEntropy`] 和它所使用的 [`EntropyRng`] 来源 都应该是安全的。
此外， [`ThreadRng`] 是一个 CSPRNG ，所以它可以作为其他算法的种子
（虽然说安全应用应该优先使用全新的/外部的熵）。

更进一步说，一个 CSPRNG 的内部状态显然必须保密。
牢记这点之后，Rand 提供的实现并不直接接触大多数内部状态，
而且 `Debug` 实现不会打印任何内部状态。
这些还不足以保护 CSPRNGs 的状态；同一个进程内的代码可能读取状态的的内存，
Rand 出于方便，允许复制和序列化 CSPRNGs 。
甚至，运行中的进程可能被操作系统复制 (fork) ，这可能使两个进程具有同一生成器的复制品。

### 非加密级库

必须强调的一点是，Rand 不是一个达到加密级别库 (cryptography library)，
虽然 Rand 采取一些方式来提供安全的随机数，但它不一定会采取所有推荐做法。
而且像加密 (encryption) 和验证 (authentication) 这样的加密过程 (cryptographic processes)
十分复杂，必须非常小心地实现才能避免纰漏和抵抗已知的攻击。
所以建议尽可能使用专门用于加密领域的库，比如 [openssl] 、 [ring] 和 [RustCrypto libraries] 。

## 额外特性

某些 PRNGs 可能提供一些额外的特性，比如：

- 支持多流，因而可以完成并行任务
- 可以在随机数流中跳转或寻找；具有很大的周期，从而可用来替代流。

## 延伸阅读

关于 PRNGs 还有很多可以说的。
这篇 [PCG paper] 非常容易理解，而且解释了更多概念。

另一篇关于 RNG 质量的 paper 是 
[好的 RNG 很（不）容易发现](https://web.archive.org/web/20160801142711/http://random.mat.sbg.ac.at/results/peter/A19final.pdf)[^RNG-paper]

[^RNG-paper]: *Good random number generators are (not so) easy to find* 

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
