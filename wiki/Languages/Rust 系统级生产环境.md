# Rust 系统与生产环境

> 参考来源：[Rust API Guidelines](https://rust-lang.github.io/api-guidelines/)、[The Rustonomicon](https://doc.rust-lang.org/nomicon/)、[thiserror](https://docs.rs/thiserror/)、[anyhow](https://docs.rs/anyhow/)、tokio 文档

---

## 4.1 错误处理模式

### 核心原则

- **`thiserror` vs `anyhow`**：
  - `thiserror`：库代码，定义结构化错误类型
  - `anyhow`：应用程序代码，快速错误处理
- **`#[from]` 自动转换**：从底层错误自动转换
- **`.context()` 添加上下文**：在传播过程中丰富错误信息
- **panic 是契约违反，不是错误**：`unwrap()` 应附注释说明不可达原因

```rust
use thiserror::Error;

#[derive(Error, Debug)]
enum AppError {
    #[error("IO error: {0}")]
    Io(#[from] std::io::Error),
    
    #[error("Parse error: {0}")]
    Parse(#[from] serde_json::Error),
    
    #[error("Invalid configuration: {field}")]
    InvalidConfig { field: String },
}

// anyhow 在应用代码中
use anyhow::{Context, Result};

fn load_config(path: &str) -> Result<Config> {
    let content = fs::read_to_string(path)
        .with_context(|| format!("Failed to read config from {}", path))?;
    
    let config: Config = serde_json::from_str(&content)
        .context("Failed to parse config")?;
    
    Ok(config)
}

// unwrap 附注释
let port = env::var("PORT")
    .expect("PORT must be set in environment"); // 程序启动必需
```

---

## 4.2 序列化、零拷贝与二进制数据

### 核心原则

- **serde 属性精确控制**：`rename`、`skip`、`default`、`flatten`
- **零拷贝反序列化**：`&str`、`&[u8]` 代替 `String`、`Vec<u8>`
- **Cow<'_, str>**：反序列化时零拷贝，修改时自动克隆
- **`bytes::Bytes`**：引用计数字节容器，支持切片不拷贝
- **zerocopy / bytemuck**：安全地将字节切片转换为结构体引用

```rust
use serde::Deserialize;

// 零拷贝反序列化
#[derive(Deserialize)]
struct Packet<'a> {
    #[serde(borrow)]
    header: &'a str,     // 借用输入数据，零拷贝
    #[serde(default)]
    payload: Option<&'a [u8]>,
}

// Cow：读时零拷贝，写时克隆
#[derive(Deserialize)]
struct Config<'a> {
    #[serde(default = "default_name")]
    name: Cow<'a, str>,
}

fn default_name() -> Cow<'static, str> {
    Cow::Borrowed("default")
}

// bytes::Bytes：网络数据的理想选择
use bytes::Bytes;

fn process_packet(data: Bytes) {
    let header = data.slice(0..4);  // 零拷贝切片
    let body = data.slice(4..);      // 引用计数，共享底层数据
}
```

---

## 4.3 Unsafe Rust — 可控的危险

### 核心原则

- **安全抽象是目标**：`unsafe` 实现细节应封装在安全 API 后
- **最小化 unsafe 块**：仅将真正需要 unsafe 的操作放入 `unsafe {}`
- **不变量文档化**：注释说明调用者/实现者必须维护的假设
- **优先使用 safe 替代方案**：`MaybeUninit` 代替未初始化内存操作
- **FFI 边界检查**：C 返回的指针必须验证后再解引用
- **packed struct 注意对齐**：`#[repr(packed)]` 字段可能未对齐

```rust
/// # Safety
/// `ptr` 必须指向一个有效的、已初始化的 `T`
/// 且调用后将接管该内存的所有权
unsafe fn take_ownership<T>(ptr: *mut T) -> T {
    // SAFETY: 调用者保证 ptr 有效且已初始化
    ptr.read()
}

// Arena/Slab 分配器：批量内存管理
pub struct Arena<T> {
    chunks: Vec<Vec<MaybeUninit<T>>>,
}

impl<T> Arena<T> {
    pub fn alloc(&mut self, value: T) -> &mut T {
        // 安全地将值放入 MaybeUninit，然后返回引用
    }
}
```

---

## 4.4 宏 — 编写代码的代码

### 核心原则

- **macro_rules! 用于简单代码生成**：模式匹配、重复 (`$()*`)
- **tt munching**：递归宏处理变长参数
- **`$crate`**：在宏中引用当前 crate 的项
- **过程宏 (proc macros)**：复杂代码生成使用 `syn` + `quote`
- **derive 宏**：自动实现 trait
- **attribute 宏**：修改/装饰函数/结构体

```rust
// macro_rules! 示例：vec! 风格宏
macro_rules! map {
    ($($key:expr => $value:expr),* $(,)?) => {{
        let mut m = ::std::collections::HashMap::new();
        $(m.insert($key, $value);)*
        m
    }};
}

// tt munching：递归处理参数
macro_rules! count_args {
    () => { 0 };
    ($head:tt $($tail:tt)*) => { 1 + count_args!($($tail)*) };
}

// 过程宏框架
use proc_macro::TokenStream;
use quote::quote;
use syn::{parse_macro_input, DeriveInput};

#[proc_macro_derive(MyTrait)]
pub fn derive_my_trait(input: TokenStream) -> TokenStream {
    let input = parse_macro_input!(input as DeriveInput);
    let name = input.ident;
    
    quote! {
        impl MyTrait for #name {
            fn do_something(&self) {}
        }
    }
    .into()
}
```

---

## 4.5 测试与基准测试模式

### 核心原则

- **单元测试**：每个模块的 `#[cfg(test)]` 模块，测试私有函数
- **集成测试**：`tests/` 目录，测试公共 API
- **文档测试**：代码示例作为测试运行
- **属性测试 (proptest)**：自动生成随机输入验证属性
- **基准测试 (criterion)**：统计稳健的基准测试，检测回归
- **基于 trait 的 mock**：通过 trait 实现测试替身

```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_add() {
        assert_eq!(add(2, 3), 5);
    }

    #[test]
    #[should_panic(expected = "overflow")]
    fn test_add_overflow() {
        add(u32::MAX, 1);
    }
}

// 属性测试
use proptest::prelude::*;

proptest! {
    #[test]
    fn test_reverse_reverse(a: Vec<i32>) {
        let mut b = a.clone();
        b.reverse();
        b.reverse();
        prop_assert_eq!(a, b);
    }
}

// 基于 trait 的 mock
trait Database {
    fn get_user(&self, id: u64) -> Option<User>;
}

struct MockDb {
    users: HashMap<u64, User>,
}

impl Database for MockDb {
    fn get_user(&self, id: u64) -> Option<User> {
        self.users.get(&id).cloned()
    }
}
```

---

## 4.6 Crate 架构与 API 设计

### 核心原则

- **Sealed Trait**：通过私有 supertrait 限制外部实现
- **`#[non_exhaustive]`**：标记枚举/结构体将来会扩展，防止破坏性变更
- **`impl Into<T>` 参数**：提高 API 易用性，自动转换
- **`impl AsRef<Path>`**：接受多种路径类型
- **`TryFrom` 解析**：从原始类型安全转换，集中验证逻辑
- **Feature flags**：条件编译控制可选功能
- **Workspace 结构**：大型项目拆分为多个 crate

```rust
// Sealed trait
mod private {
    pub trait Sealed {}
}

#[non_exhaustive]
pub enum Error {
    NotFound,
    InvalidInput,
    // 将来可以安全添加新变体
}

// 灵活的参数类型
pub fn open(path: impl AsRef<Path>) -> Result<File> {
    std::fs::File::open(path.as_ref())
}

pub fn set_timeout(duration: impl Into<Duration>) {
    let duration = duration.into();
    // ...
}

// TryFrom 集中验证
#[derive(Debug)]
pub struct Port(u16);

impl TryFrom<u16> for Port {
    type Error = &'static str;
    fn try_from(value: u16) -> Result<Self, Self::Error> {
        if value == 0 {
            return Err("port cannot be 0");
        }
        Ok(Self(value))
    }
}

// Feature flags
#[cfg(feature = "tokio")]
pub mod async_impl;

#[cfg(not(feature = "tokio"))]
pub mod sync_impl;
```

---

## 4.7 Async/Await 精要

### 核心原则

- **`tokio` 作为异步运行时标准**：生态最成熟
- **`spawn_blocking`**：将 CPU 密集型任务移出异步线程池
- **`join!` / `select!`**：并发等待多个 future
- **`Send` 约束**：跨 `.await` 的数据必须实现 `Send`
- **`tokio::sync::Mutex`**：跨 await 持有的锁（std::sync::Mutex 在 await 期间持有会阻塞线程池）
- **避免在异步函数中使用阻塞 IO**

```rust
use tokio::time::{sleep, Duration};

async fn fetch_data() -> Result<String> {
    // spawn_blocking：CPU 密集型任务
    let result = tokio::task::spawn_blocking(|| {
        expensive_computation()
    }).await?;
    
    Ok(result)
}

// join! 并发执行
let (a, b) = tokio::join!(
    fetch_from_api1(),
    fetch_from_api2(),
);

// select! 等待第一个完成
tokio::select! {
    result = fetch_from_api() => println!("API: {:?}", result),
    _ = sleep(Duration::from_secs(5)) => println!("timeout"),
}

// tokio::sync::Mutex：跨 await 持有
async fn update_cache(cache: &tokio::sync::Mutex<Cache>, key: String, value: String) {
    let mut guard = cache.lock().await; // 异步等待锁
    guard.insert(key, value);
    // 锁在 guard  drop 时释放
}
```

---

## 4.8 可观测性与生产遥测

### 核心原则

- **日志、指标、链路追踪三位一体**：日志解释事件，指标衡量严重度，trace 定位跨服务链路。
- **优先使用 `tracing`**：用 Span 传递结构化上下文，避免在每条日志中重复手写 request id。
- **避免 Span guard 跨 `.await`**：异步代码中使用 `#[instrument]` 或 `.instrument()`，不要手动 `span.enter()` 后等待 Future。
- **生产环境限制日志级别**：发布版可用 `max_level_info` 等编译期 feature 移除 debug/trace。
- **优雅关闭时 flush 遥测**：OTLP/trace provider 在进程退出前需要 shutdown，避免尾部数据丢失。

详见 [[Rust 可观测性]]。

---

## 反模式

### 错误处理

| 反模式 | 问题 | 解决方案 |
|--------|------|----------|
| `unwrap()` 无注释 | 难以判断是否真的不可达 | 使用 `expect("原因")` |
| 使用 `panic!` 处理可恢复错误 | 栈展开开销 | 返回 `Result` |
| 丢失错误上下文 | 难以定位问题 | 使用 `.context()` 或自定义错误类型 |
| `Result` 嵌套过深 | 代码难以阅读 | 使用 `?` 和 `and_then` 链 |

### 性能

| 反模式 | 问题 | 解决方案 |
|--------|------|----------|
| 运行期计算常量 | 运行时开销 | 使用 `const fn` |
| 不必要的 `String` 分配 | 堆分配开销 | 使用 `&str` 或 `Cow` |
| 复制大数据结构 | 内存和 CPU 开销 | 使用引用或 `Arc` |
| 忽略迭代器优化 |  missed SIMD/向量化 | 优先使用迭代器 |

---

## 参考资源

### 官方资源

- [Rust API Guidelines](https://rust-lang.github.io/api-guidelines/) — API 设计官方指南
- [Cargo Workspaces](https://doc.rust-lang.org/cargo/reference/workspaces.html) — 工作空间文档
- [Rust Reference](https://doc.rust-lang.org/reference/) — 语言参考

### 社区资源

- [The Rustonomicon](https://doc.rust-lang.org/nomicon/) — Unsafe Rust 黑魔法书
- [Rust for Rustaceans](https://rust-for-rustaceans.com/) — 高级 Rust 编程
- [Rust Atomics and Locks](https://marabos.nl/atomics/) — 并发编程深度指南

### 常用 crate

| 用途 | 推荐 crate |
|------|-----------|
| 异步运行时 | `tokio` |
| 错误处理（库） | `thiserror` |
| 错误处理（应用） | `anyhow` |
| 序列化 | `serde` |
| 零拷贝序列化 | `zerocopy`, `bytemuck` |
| 属性测试 | `proptest` |
| 基准测试 | `criterion` |
| 并发通道 | `crossbeam-channel` |
| 数据并行 | `rayon` |
| 字节处理 | `bytes` |
| 日志/追踪 | `tracing` |
| 过程宏 | `syn`, `quote` |

---

## 相关笔记

- [[Rust]] — Rust 核心最佳实践与概述
- [[Rust 可观测性]] — tracing、metrics、OpenTelemetry 与生产遥测
