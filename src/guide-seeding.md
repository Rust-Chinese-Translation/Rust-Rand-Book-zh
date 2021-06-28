# 随机数种子

PRNGs 的输出值由初始状态决定。
有些 PRNG 从定义中指明了初始状态应该如何从秘钥 (key) 生成，
初始状态通常作为字节序列来传递给加密级生成器或者以一个字 (word) 传递给小型的 PRNGs 。
Rand 用 [`SeedableRng`] trait 来把这种形式统一起来，从而适用于 Rand 定义的生成器。

## 种子类型

Rand 要求所有支持种子的 RNGs 需定义一个 [`Seed`] 关联类型，
这个类型需满足 `AsMut<[u8]> + Default + Sized` 。

通常对于生成固定的 `N` 个随机值则使用  `[u8; N]` 类型：
对于非加密级的 PRNGs 建议使用 `[u8; 12]` 或更大长度的数组；
对于 CSPRNGs 则建议使用 `[u8; 32]` 。

PRNGs 可以直接使用 [`SeedableRng::from_seed`] 来设置种子的值。

## 种子的来源

### 全新的熵

使用 [`SeedableRng::from_entropy`] 可以很容易地直接从操作系统设置新的种子。

```rust,editable
use rand::prelude::*;
use rand_chacha::ChaCha20Rng;

let rng = ChaCha20Rng::from_entropy();
```

注意，这需要让 `rand_core` 开启 `getrandom` feature 。

### 另一个 RNG

很明显，另一个 RNG 也可以作为种子。
Rand 为此提供了一个很方便的方法：

```rust,editable
use rand::prelude::*;
use rand_pcg::Pcg64;

let rng = Pcg64::from_rng(thread_rng());
```

但是，如果说你想保存秘钥，然后之后再使用秘钥，那么，你需要更显式地指明：

```rust,editable
use rand::prelude::*;
use rand_chacha::ChaCha8Rng;

let mut seed: <ChaCha8Rng as SeedableRng>::Seed = Default::default();
thread_rng().fill(&mut seed);
let rng = ChaCha8Rng::from_seed(seed);
```

**警告**：
有些简单的 PRNGs，尤其是 [`XorShiftRng`] ，
给它们设置同类 PRNG 的种子时，效果并不好
（比如这种情况下， Xorshift 会生成一份复制品）。
对于加密级的 PRNGs ，设置同类 PRNG 没什么问题；
但是对应非加密级的 PRNGs ，建议从不同的 PRNG 生成种子。
对于确定性的主生成器 (deterministic master generator)，
使用 [`ChaCha8Rng`] 是个不错的选中。
在加密场景，则建议 [`ChaCha12Rng`] 或更高的版本。

[`ChaCha12Rng`]:https://rust-random.github.io/rand/rand_chacha/struct.ChaCha12Rng.html

### 一个简单的数字

有些应用，尤其是模拟领域，你所需要的只是一串确切的、固定的随机数种子，
比如 1, 2, 3 等。

那么 [`SeedableRng::seed_from_u64`] 就是专门为这种情况提供给你的。
其背后使用一个简单的 PRNG 来从输入值填充种子的 bits ，
从而让两个相近的数字，比如 0 和 1 分别产生完全不同且独立的 RNG 序列。

```rust,editable
use rand::prelude::*;
use rand_pcg::Pcg64;

let rng = Pcg64::seed_from_u64(2);
```

注意，一个 64-bits 或低于 64-bits 的数字不可能保证安全，
所以这不应该应用在加密领域或者赌博游戏上。

### 字符串或散列值

当你让用户输入字符串从而给 RNG 设置种子时，
理想情况下字符串的所有部分应该都会影响 RNG ，
而且对这个字符串做出很小的更改都应该产生完全独立的随机数序列。

这可以通过使用 hash 函数，压缩所有输入数据到一个散列值，
然后把这个值用于给 RNG 提供种子。
[`rand_seeder`] crate 就是专门为这种情况设计的。

```rust,editable
use rand::prelude::*;
use rand_seeder::{Seeder, SipHasher};
use rand_pcg::Pcg64;

// In one line:
let rng: Pcg64 = Seeder::from("stripy zebra").make_rng();

// If we want to be more explicit, first we create a SipRng:
let hasher = SipHasher::from("a sailboat");
let mut hasher_rng = hasher.into_rng();
// (Note: hasher_rng is a full RNG and can be used directly.)

// Now, we use hasher_rng to create a seed:
let mut seed: <Pcg64 as SeedableRng>::Seed = Default::default();
hasher_rng.fill(&mut seed);

// And create our RNG from that seed:
let rng = Pcg64::from_seed(seed);
```

注意 `rand_seeder` **不适合** 加密级场景，
它也不提供密码散列 (password hasher) ，
所以对于秘钥衍生 (key-derivation) 的应用，请使用 [Argon2] 这样的 crate 。

[Argon2]:https://docs.rs/argon2/*/argon2/

[`SeedableRng`]: https://rust-random.github.io/rand/rand_core/trait.SeedableRng.html
[`Seed`]: https://rust-random.github.io/rand/rand_core/trait.SeedableRng.html#type.Seed
[`SeedableRng::from_seed`]: https://rust-random.github.io/rand/rand_core/trait.SeedableRng.html#tymethod.from_seed
[`SeedableRng::from_rng`]: https://rust-random.github.io/rand/rand_core/trait.SeedableRng.html#method.from_rng
[`SeedableRng::seed_from_u64`]: https://rust-random.github.io/rand/rand_core/trait.SeedableRng.html#method.seed_from_u64
[`SeedableRng::from_entropy`]: https://rust-random.github.io/rand/rand_core/trait.SeedableRng.html#method.from_entropy
[`XorShiftRng`]: https://rust-random.github.io/rand/rand_xorshift/struct.XorShiftRng.html
[`ChaCha8Rng`]: https://rust-random.github.io/rand/rand_chacha/struct.ChaCha8Rng.html
[`rand_seeder`]: https://github.com/rust-random/seeder/
