# Rust 并发与运行时

> 参考来源：[The Rust Programming Language Book](https://doc.rust-lang.org/book/)、[Rust Atomics and Locks](https://marabos.nl/atomics/)、crossbeam 文档、tokio 文档

---

## 3.1 通道与消息传递

### 核心原则

- **优先使用 bounded channel**：无限通道可能导致内存无限增长
- **crossbeam-channel 作为标准库替代**：多生产者多消费者，更好的性能
- **Actor 模式**：每个 actor 拥有独立状态，通过消息通信
- **select! 处理多路事件**：同时等待多个通道操作
- **rendezvous channel**：零容量通道，强制发送者和接收者同步

```rust
use crossbeam_channel::{bounded, select, unbounded};

// Bounded channel：背压机制防止生产者过快
let (tx, rx) = bounded(100);

// Actor 模式示例
struct Actor<T> {
    tx: Sender<T>,
}

impl<T: Send + 'static> Actor<T> {
    fn new<F: FnMut(T) + Send + 'static>(mut handler: F) -> Self {
        let (tx, rx) = bounded(100);
        std::thread::spawn(move || {
            while let Ok(msg) = rx.recv() {
                handler(msg);
            }
        });
        Self { tx }
    }
    fn send(&self, msg: T) {
        let _ = self.tx.send(msg);
    }
}

// select! 多路复用
select! {
    recv(rx1) -> msg => println!("from rx1: {:?}", msg),
    recv(rx2) -> msg => println!("from rx2: {:?}", msg),
    default(Duration::from_millis(100)) => println!("timeout"),
}
```

---

## 3.2 并发 vs 并行 vs 线程

### 核心原则

- **并发 ≠ 并行**：并发是结构（同时处理多任务），并行是执行（同时运行多任务）
- **std::thread::scope**：允许线程借用非 'static 数据
- **数据并行使用 rayon**：自动将迭代器工作负载分配到线程池
- **同步原语选择**：
  - `std::sync::Mutex`：跨线程共享可变状态
  - `std::sync::RwLock`：读多写少的场景
  - `std::sync::atomic`：简单的计数器、标志位
- **初始化原语**：`std::sync::OnceLock` / `std::sync::LazyLock`（Rust 1.80+）

```rust
// thread::scope：借用局部数据
let mut data = vec![1, 2, 3];
std::thread::scope(|s| {
    s.spawn(|| {
        data.push(4); // ✅ 可以借用 data
    });
});

// rayon 数据并行
use rayon::prelude::*;
let sum: i32 = (0..1_000_000)
    .into_par_iter()
    .map(|x| x * x)
    .sum();

// OnceLock 延迟初始化
static CONFIG: OnceLock<Config> = OnceLock::new();
fn get_config() -> &'static Config {
    CONFIG.get_or_init(|| Config::load())
}
```

---

## 3.3 闭包与高阶函数

### 核心原则

- **Fn/FnMut/FnOnce 层次**：
  - `Fn`：可多次不可变借用（只读环境）
  - `FnMut`：可多次可变借用（可修改环境）
  - `FnOnce`：只能调用一次（会消耗环境）
- **捕获策略**：闭包自动选择最小捕获方式（Rust 2024 edition 改进）
- **高阶 API 设计**：接受闭包增加 API 灵活性
- **with-pattern**：接受闭包配置对象，避免部分构造

```rust
// with-pattern
pub struct Builder {
    name: String,
    timeout: Duration,
}

impl Builder {
    pub fn with_name(mut self, name: impl Into<String>) -> Self {
        self.name = name.into();
        self
    }
    pub fn with_timeout(mut self, timeout: Duration) -> Self {
        self.timeout = timeout;
        self
    }
    pub fn build(self) -> Client {
        Client { name: self.name, timeout: self.timeout }
    }
}

// 使用
let client = Builder::default()
    .with_name("api-client")
    .with_timeout(Duration::from_secs(30))
    .build();
```

---

## 3.4 函数式 vs 命令式：优雅胜出之时

### 核心原则

- **优先使用组合子**：`map`、`filter`、`and_then` 等使代码更声明式
- **迭代器链优于显式循环**：更短、更可组合、更不容易出边界错误
- **决策框架**：
  - 单次操作 + 副作用 → 命令式
  - 数据转换管道 → 函数式
  - 错误传播链 → 函数式 + `?`

```rust
// 函数式：数据转换管道
let active_users: Vec<_> = users
    .into_iter()
    .filter(|u| u.is_active)
    .map(|u| u.email)
    .collect();

// 函数式错误处理链
let content = fs::read_to_string(path)?
    .lines()
    .find(|line| line.starts_with("config:"))
    .map(|line| line.trim_start_matches("config:").trim())
    .ok_or("config not found")?;

// 命令式：单次 IO 操作
let mut file = File::create("output.txt")?;
file.write_all(b"hello")?;
file.flush()?;
```

---

## 3.5 智能指针与内部可变性

### 核心原则

- **`Box<T>`**：堆分配，固定大小 trait object
- **`Rc<T>`**：单线程引用计数，共享所有权
- **`Arc<T>`**：多线程引用计数，共享所有权
- **`Weak<T>`**：弱引用，打破循环引用
- **`Cell<T>`**：单线程内部可变性（只能用于 Copy 类型）
- **`RefCell<T>`**：单线程运行时借用检查
- **`Cow<T>`**：写时克隆，读时零拷贝
- **`Pin<T>`**：保证内存地址不变，用于 async/自引用结构
- **`ManuallyDrop<T>`**：手动控制 drop 时机

```rust
// Cow：避免不必要的克隆
fn greet(name: &str) -> Cow<str> {
    if name.is_empty() {
        "World".into() // 借用静态字符串
    } else {
        format!("Hello, {}", name).into() // 拥有新字符串
    }
}

// 自引用结构需要 Pin
use std::pin::Pin;
use std::marker::PhantomPinned;

struct SelfReferential {
    data: String,
    ptr_to_data: *const String,
    _pin: PhantomPinned,
}

// 只能通过 Pin<&mut SelfReferential> 访问
```

---

## 反模式

| 反模式 | 问题 | 解决方案 |
|--------|------|----------|
| 无限（unbounded）通道 | 内存无限增长 | 使用 bounded channel |
| 在异步代码中使用阻塞 IO | 阻塞线程池 | 使用 tokio 的异步 IO |
| `std::sync::Mutex` 跨越 `.await` | 阻塞其他任务 | 使用 `tokio::sync::Mutex` |
| 忘记处理 `Weak` 引用 | 循环引用导致内存泄漏 | 在父子关系中使用 `Weak` |
| 过度使用 `Arc<Mutex<T>>` | 性能瓶颈 | 考虑消息传递或原子操作 |

---

## 参考资源

- [The Rust Programming Language Book — Concurrency](https://doc.rust-lang.org/book/ch16-00-concurrency.html)
- [Rust Atomics and Locks](https://marabos.nl/atomics/) — 并发编程深度指南
- [crossbeam-channel 文档](https://docs.rs/crossbeam-channel/)
- [rayon 文档](https://docs.rs/rayon/)

---

## 相关笔记

- [[Rust]] — Rust 核心最佳实践与概述
