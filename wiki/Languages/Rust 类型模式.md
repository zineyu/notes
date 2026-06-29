---
tags:
  - Languages
---

# Rust 类型层面模式

> 参考来源：[Rust API Guidelines](https://rust-lang.github.io/api-guidelines/)、[The Rust Programming Language Book](https://doc.rust-lang.org/book/)、[Rust Patterns Book](https://rust-unofficial.github.io/patterns/)

---

## 2.1 泛型 — 全景解析

### 核心原则

- **单态化是有代价的**：每个泛型实例都会产生独立的机器码，注意代码体积膨胀
- **优先用关联类型而非泛型参数**表达类型之间的关系
- **利用 const generics** 在类型级别编码数值信息
- **用 `const fn` 在编译期计算**

### 代码示例

```rust
// BAD: 泛型参数过多导致 API 冗长
fn process<T, U, V, W>(a: T, b: U, c: V) -> W

// GOOD: 关联类型减少参数数量
trait Processor {
    type Input;
    type Output;
    fn process(&self, input: Self::Input) -> Self::Output;
}

// const generics：编译期已知大小的数组包装器
struct Buffer<T, const N: usize> {
    data: [T; N],
}

impl<T: Default, const N: usize> Buffer<T, N> {
    const fn new() -> Self {
        Self { data: [(); N].map(|_| T::default()) }
    }
}

// const fn：编译期计算缓冲区大小
const fn next_power_of_two(n: usize) -> usize {
    let mut p = 1;
    while p < n {
        p *= 2;
    }
    p
}

const BUF_SIZE: usize = next_power_of_two(100); // 128
```

### 常见反模式

| 反模式 | 问题 | 解决方案 |
|--------|------|----------|
| 泛型参数过多 | API 难以使用 | 使用关联类型 |
| 忽视单态化开销 | 二进制膨胀 | 使用 `&dyn Trait` 或枚举分发 |
| 运行期计算常量 | 运行时开销 | 使用 `const fn` 和 `const` |

---

## 2.2 深入理解 Trait

### 核心原则

- **关联类型 vs 泛型参数**：关联类型表达"一一对应"关系，泛型参数表达"多对多"关系
- **泛型关联类型 (GATs)**：允许关联类型携带自己的生命周期和泛型参数
- **Supertrait**：通过 `trait Sub: Super` 要求实现 Sub 必须先实现 Super
- **Blanket Implementation**：为所有满足条件的类型自动实现 trait
- **Marker Trait**：零方法的 trait，仅用于类型标记（如 `Send`、`Sync`）
- **Trait Object Safety**：只有满足特定条件的 trait 才能用于 `dyn Trait`
- **Extension Trait**：为外部类型添加方法，避免包装类型
- **Sealed Trait**：限制 trait 的实现者，保证向后兼容性

### 代码示例

```rust
// 关联类型 vs 泛型参数
trait Iterator {
    type Item; // 一个迭代器只产生一种 Item
    fn next(&mut self) -> Option<Self::Item>;
}

trait ConvertTo<T> { // 一个类型可转换为多种目标类型
    fn convert(&self) -> T;
}

// GAT：泛型关联类型
trait LendingIterator {
    type Item<'a> where Self: 'a;
    fn next<'a>(&'a mut self) -> Option<Self::Item<'a>>;
}

// Supertrait
trait Drawable: Debug + Clone {
    fn draw(&self);
}

// Blanket Implementation
impl<T: Display> ToString for T {
    fn to_string(&self) -> String {
        // ...
    }
}

// Extension Trait
pub trait StrExt {
    fn is_ascii_alphanumeric_only(&self) -> bool;
}

impl StrExt for str {
    fn is_ascii_alphanumeric_only(&self) -> bool {
        self.bytes().all(|b| b.is_ascii_alphanumeric())
    }
}

// Sealed Trait
trait Sealed {} // private trait
mod private {
    pub trait Sealed {}
}

pub trait PublicTrait: private::Sealed {
    fn public_method(&self);
}
```

### Trait Object Safety 规则

- 方法返回 `Self` → 不安全（vtable 不知道大小）
- 方法有泛型参数 → 不安全（单态化无法通过 vtable 实现）
- 关联常量 → 不安全（vtable 只存储方法）

```rust
// Object-Safe
trait Safe {
    fn method(&self, x: &dyn Any) -> Box<dyn Any>;
}

// NOT Object-Safe
trait Unsafe {
    fn method(&self) -> Self;        // 返回 Self
    fn generic<T>(&self, x: T);     // 泛型方法
    const CONST: usize;             // 关联常量
}
```

---

## 2.3 Newtype 与 Type-State 模式

### Newtype 模式

- **零成本类型安全**：包装基本类型，在编译时防止混淆
- **控制暴露**：通过 Deref 选择性暴露内部方法，但要谨慎

```rust
#[derive(Debug, Clone, Copy, PartialEq, Eq, Hash, Default)]
pub struct UserId(u64);

#[derive(Debug, Clone, Copy, PartialEq, Eq, Hash, Default)]
pub struct OrderId(u64);

// 编译错误: fetch_user(OrderId(123)) -> 类型不匹配!

// Deref 决策：仅在语义完全相同时使用
// ✅ 为 Kilometers(u32) Deref 到 u32
// ❌ 不要为 UserId Deref 到 u64（丢失语义保护）
```

### Type-State 模式

- **编译时协议验证**：将状态编码到类型中，非法状态不可表示
- **状态转移 = 类型转换**：消耗旧状态，返回新状态

```rust
// 编译时确保：未连接 → 已连接 → 已认证
struct Unconnected;
struct Connected { socket: TcpStream };
struct Authenticated { socket: TcpStream, token: AuthToken };

struct Client<State> {
    state: State,
}

impl Client<Unconnected> {
    fn new() -> Self {
        Self { state: Unconnected }
    }
    fn connect(self, addr: &str) -> Result<Client<Connected>, io::Error> {
        let socket = TcpStream::connect(addr)?;
        Ok(Client { state: Connected { socket } })
    }
}

impl Client<Connected> {
    fn authenticate(self, creds: Credentials) -> Result<Client<Authenticated>, io::Error> {
        let token = authenticate(self.state.socket, creds)?;
        Ok(Client { state: Authenticated { socket: self.state.socket, token } })
    }
}

impl Client<Authenticated> {
    fn send_data(&mut self, data: &[u8]) -> Result<(), io::Error> {
        // 只有在认证后才能发送数据
        self.state.socket.write_all(data)
    }
}

// 编译时保证：无法在未认证状态下发送数据
// let client = Client::new().connect("...")?;
// client.send_data(b"..."); // 编译错误！
```

---

## 2.4 PhantomData — 不承载数据的类型

- **生命周期标记**：告诉编译器类型"逻辑上"包含某个生命周期
- **单位测量**：编译时类型安全的物理单位
- **方差控制**：控制泛型参数在子类型关系中的行为
- **Drop Check**：标记类型拥有特定类型的所有权（影响 drop checker）

```rust
use std::marker::PhantomData;

// 生命周期标记：确保指针不会超过原始切片
struct Slice<'a, T> {
    ptr: *const T,
    len: usize,
    _marker: PhantomData<&'a T>, // 逻辑上借用 'a
}

// 单位测量
type Meters = f64;
type Seconds = f64;

struct Quantity<T>(f64, PhantomData<T>);

// 编译时防止单位混淆
fn distance(t: Quantity<Seconds>, v: Quantity<Meters>) -> Quantity<Meters> {
    Quantity(t.0 * v.0, PhantomData)
}

// 方差控制
struct Covariant<T>(PhantomData<T>);      // T 协变
struct Contravariant<T>(PhantomData<fn(T)>); // T 逆变
struct Invariant<T>(PhantomData<fn(T) -> T>); // T 不变
```

---

## 反模式

| 反模式 | 问题 | 解决方案 |
|--------|------|----------|
| 泛型参数过多 | API 难以使用 | 使用关联类型 |
| 忽视单态化开销 | 二进制膨胀 | 使用 `&dyn Trait` 或枚举分发 |
| 过度克隆以绕过借用检查器 | 不必要的分配 | 使用引用 `&T` |
| 滥用 `Deref` 实现 | 丢失语义保护 | 仅在语义完全相同时使用 |
| 在简单借用足够时使用 `Rc`/`Arc` | 引用计数开销 | 简单引用 |

---

## 参考资源

- [Rust API Guidelines](https://rust-lang.github.io/api-guidelines/) — API 设计官方指南
- [The Rust Programming Language Book](https://doc.rust-lang.org/book/) — Rust 官方教程
- [Rust Patterns Book](https://rust-unofficial.github.io/patterns/) — 设计模式合集
- [Rust for Rustaceans](https://rust-for-rustaceans.com/) — 高级 Rust 编程

---

## 相关笔记

- [[Rust]] — Rust 核心最佳实践与概述
