# 随机序列

Rand 通过 [`IteratorRandom`] 和 [`SliceRandom`] 两个 traits 实现了某些常见的随机序列操作。

-   `choose` 从序列中均匀抽取一个元素。
-   `choose_multiple` 从序列中无放回地均匀抽取元素。
-   `choose_weighted` 从切片中按照指定的权重非均匀地抽取一个元素。
    （也可参考 [`WeightedIndex`] ）
-   `shuffle` 打乱切片元素的顺序。
-   `partial_shuffle` 可以部分打乱切片，高效地抽取 `amount` 个顺序随机的元素。

[`IteratorRandom`]: https://rust-random.github.io/rand/rand/seq/trait.IteratorRandom.html
[`SliceRandom`]: https://rust-random.github.io/rand/rand/seq/trait.SliceRandom.html
[`WeightedIndex`]: https://rust-random.github.io/rand/rand/distributions/struct.WeightedIndex.html
