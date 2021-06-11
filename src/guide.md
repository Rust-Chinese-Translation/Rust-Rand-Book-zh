# 使用指南

这一章目的是解释 Rand 库的一些概念。

1.  [什么是随机数据和随机性？](guide-data.md)
1.  [有哪些类型的随机数生成器](guide-gen.md)
1.  [Rand 提供哪些随机数生成器](guide-rngs.md)
1.  [如何把随机数据变成有用的随机值](guide-values.md)
1.  [概率分布：更方便掌控随机值](guide-dist.md)
1.  [序列](guide-seq.md)
1.  [处理 Error](guide-err.md)

## 导入条目

导入 Rand 里的条目，最方便的做法是使用 [`prelude`] 。
它包括 Rand 库最重要的部分，只包含 [`prelude`] 的话不太会造成同名冲突。

注意 Rand 0.5 相较于之前的版本，在模块结构和内容上进行了很大的调整。
可能有些旧名字依然被保留，但它们已不出现在文档介绍里，将来这些名字会被移除掉。
因此建议你导入时使用 prelude 或者最新的模块结构。

## 更多例子

以下案例应用可以给你一些启发：

- [对 π 的 Monte Carlo 估计](
  https://github.com/rust-random/rand/blob/master/examples/monte-carlo.rs)
- [三门问题](
   https://github.com/rust-random/rand/blob/master/examples/monty-hall.rs)

[`prelude`]: https://rust-random.github.io/rand/rand/prelude/index.html
