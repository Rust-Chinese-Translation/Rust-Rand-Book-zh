# 入门指引

## 例子

让我们从这个例子出发吧（[playground 链接][playground link]）：

[playground link]:(https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=7792ed032694bc558ca229be71a7783a)

```rust,editable
# extern crate rand;
// import commonly used items from the prelude:
use rand::prelude::*;

fn main() {
    // We can use random() immediately. It can produce values of many common types:
    let x: u8 = random();
    println!("{}", x);

    if random() { // generates a boolean
        println!("Heads!");
    }

    // If we want to be a bit more explicit (and a little more efficient) we can
    // make a handle to the thread-local generator:
    let mut rng = thread_rng();
    if rng.gen() { // random bool
        let x: f64 = rng.gen(); // random number in range [0, 1)
        let y = rng.gen_range(-10.0..10.0);
        println!("x is: {}", x);
        println!("y is: {}", y);
    }

    println!("Die roll: {}", rng.gen_range(1..=6));
    println!("Number from 0 to 9: {}", rng.gen_range(0..10));
    
    // Sometimes it's useful to use distributions directly:
    let distr = rand::distributions::Uniform::new_inclusive(1, 100);
    let mut nums = [0i32; 3];
    for x in &mut nums {
        *x = rng.sample(distr);
    }
    println!("Some numbers: {:?}", nums);

    // We can also interact with iterators and slices:
    let arrows_iter = "➡⬈⬆⬉⬅⬋⬇⬊".chars();
    println!("Lets go in this direction: {}", arrows_iter.choose(&mut rng).unwrap());
    let mut nums = [1, 2, 3, 4, 5];
    nums.shuffle(&mut rng);
    println!("I shuffled my {:?}", nums);
}
```

首先，你会注意到我们从 [prelude] 引入所有需要的东西。
这是以一种偷懒的方式来 `use` Rand ，就像标准库的 [prelude][std-prelude] 一样，
只引入最常用的条目 (items) 。
如果你不希望引入 [prelude] ，那么请一定记得引入最重要的 [`Rng`] trait ！

[std-prelude]:https://doc.rust-lang.org/std/prelude/index.html

通过使用 [`thread_rng`] 函数或者 [`random`] 函数，
Rand 库会根据需要自动初始化一个基于本地线程的 (thread-local)
安全的 [随机数生成器](guide-gen.md) (generator 或 random number generator 或 **RNG**) 。

[`random`] 函数只能以 [`Standard`] 这样依赖于类型方式来对值进行抽样，
而 [`thread_rng`] 能让你操控生成器。
所有的生成器都实现了 [`Rng`] trait ，
这个 trait 提供了上面用到的 [`gen`] 、 [`gen_range`] 和 [`sample`] 方法。

Rand 通过 [`IteratorRandom`] 和 [`SliceRandom`] 两个 trait
分别对迭代器和切片提供了随机数相关的功能。

更多例子请参考 [API 文档][API documentation] 或者本书的 [使用指南][guide] 。

## 设置生成器种子

指定种子需要指定 RNG ，然后使用 [`seed_from_u64`] 或 [`from_seed`] 方法。

注意 [`seed_from_u64`] **不适合** 加密领域，
因为单个 `u64` 无法提供足够的信息熵 (entropy) 来安全设置 RNG 种子。

我们在下面使用 `ChaCha8Rng` ，是因为它很快，而且具有良好的移植性。
到 [支持的 RNGs][RNGs] 页面进行详细阅读。
但是如果你很关心重现结果的话，就避免使用 `SmallRng` 和 `StdRng` 。

```rust,editable
extern crate rand;
extern crate rand_chacha;

use rand::{Rng, SeedableRng};

fn main() {
    let mut rng = rand_chacha::ChaCha8Rng::seed_from_u64(10);
    println!("Random f32: {}", rng.gen::<f32>());
}
```

[API documentation]: https://rust-random.github.io/rand/rand/index.html
[guide]: guide.md
[RNGs]: guide-rngs.md
[prelude]: https://rust-random.github.io/rand/rand/prelude/index.html
[`Rng`]: https://rust-random.github.io/rand/rand/trait.Rng.html
[`gen`]: https://rust-random.github.io/rand/rand/trait.Rng.html#method.gen
[`gen_range`]: https://rust-random.github.io/rand/rand/trait.Rng.html#method.gen_range
[`sample`]: https://rust-random.github.io/rand/rand/trait.Rng.html#method.sample
[`thread_rng`]: https://rust-random.github.io/rand/rand/fn.thread_rng.html
[`random`]: https://rust-random.github.io/rand/rand/fn.random.html
[`Standard`]: https://rust-random.github.io/rand/rand/distributions/struct.Standard.html
[`IteratorRandom`]: https://rust-random.github.io/rand/rand/seq/trait.IteratorRandom.html
[`SliceRandom`]: https://rust-random.github.io/rand/rand/seq/trait.SliceRandom.html
[`seed_from_u64`]: https://rust-random.github.io/rand/rand/trait.SeedableRng.html#method.seed_from_u64
[`from_seed`]: https://rust-random.github.io/rand/rand/trait.SeedableRng.html#tymethod.from_seed
