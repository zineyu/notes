# Rust 最佳实践

> 参考来源：[Rust API Guidelines](https://rust-lang.github.io/api-guidelines/)、[The Rust Programming Language Book](https://doc.rust-lang.org/book/)、[Rust Style Guide](https://doc.rust-lang.org/style-guide/)、[Rust Patterns Book](https://rust-unofficial.github.io/patterns/)、Cargo Workspaces 官方文档、Rust Atomics and Locks、Rust for Rustaceans

---

## 核心原则

### 1. 所有权与借用优先（Ownership and Borrowing First）

充分利用 Rust 的所有权系统，而非与之对抗。优先使用引用（`&` 和 `&mut`）进行只读或单写访问，避免不必要的克隆。借用检查器是设计工具，不是障碍。

### 2. 显式错误处理（Explicit Error Handling）

对所有可恢复的错误使用 `Result<T, E>` 和 `Option<T>`。使用 `?` 操作符传播错误。仅在真正不可达的情况下使用 `unwrap()` 和 `expect()`，且必须附带清晰的注释说明原因。

### 3. 零成本抽象（Zero-Cost Abstractions）

优先使用迭代器和组合子而非基于索引的循环。不需要所有权时使用 `&str` 代替 `String`。利用栈分配，避免不必要的堆分配。

### 4. 类型安全通过类型系统（Type Safety Through Type System）

使用新类型模式（Newtype）、Type-State 模式、PhantomData 在编译时提供静态区分和协议验证。积极实现常用 trait（`Debug`、`Clone`、`Default`、`Send`、`Sync`）。

### 5. 基于工作区的模块化（Workspace-Based Modularity）

将大型项目组织为 Cargo workspace，保持清晰的依赖方向（底层 crate 绝不依赖上层 crate）。在根 `Cargo.toml` 中集中管理依赖版本。

---

## 专题指南

| 主题 | 文件 | 内容 |
|------|------|------|
| **类型层面模式** | [[Rust 类型模式]] | 泛型、Trait、Newtype、Type-State、PhantomData |
| **并发与运行时** | [[Rust 并发与运行时]] | 通道与消息传递、线程与并行、闭包与高阶函数、函数式编程、智能指针 |
| **系统与生产环境** | [[Rust 系统级生产环境]] | 错误处理、序列化与零拷贝、Unsafe Rust、宏、测试、Crate 架构、Async/Await |

---

## 项目架构/目录规范

### 推荐的工作区结构

```text
myproject/
├── Cargo.toml              # 工作区清单（虚拟）
├── crates/
│   ├── core/               # 业务逻辑、领域模型（零依赖）
│   ├── types/              # 共享数据结构、序列化
│   ├── db/                 # 数据库访问层
│   ├── api/                # HTTP API 服务器
│   └── cli/                # 命令行界面
├── tools/
│   └── codegen/
├── tests/                  # 集成测试
└── examples/               # 使用示例
```

### 根 Cargo.toml 示例

```toml
[workspace]
members = ["crates/*", "tools/*"]
resolver = "2"

[workspace.package]
version = "0.1.0"
edition = "2024"
authors = ["Team <team@example.com>"]

[workspace.dependencies]
tokio = { version = "1.0", features = ["full"] }
serde = { version = "1.0", features = ["derive"] }
tracing = "0.1"
thiserror = "1.0"
anyhow = "1.0"
```

---

## 常见反模式汇总

### 类型系统

| 反模式 | 问题 | 解决方案 |
|--------|------|----------|
| 泛型参数过多 | API 难以使用 | 使用关联类型 |
| 忽视单态化开销 | 二进制膨胀 | 使用 `&dyn Trait` 或枚举分发 |
| 过度克隆以绕过借用检查器 | 不必要的分配 | 使用引用 `&T` |
| 在生产代码中使用 `unwrap()` | Panic 导致崩溃 | 使用 `?` 或 `match` 处理 `Result` |
| 在简单借用足够时使用 `Rc`/`Arc` | 引用计数开销 | 简单引用 |
| 滥用 `Deref` 实现 | 丢失语义保护 | 仅在语义完全相同时使用 |

### 并发

| 反模式 | 问题 | 解决方案 |
|--------|------|----------|
| 无限（unbounded）通道 | 内存无限增长 | 使用 bounded channel |
| 在异步代码中使用阻塞 IO | 阻塞线程池 | 使用 tokio 的异步 IO |
| `std::sync::Mutex` 跨越 `.await` | 阻塞其他任务 | 使用 `tokio::sync::Mutex` |
| 忘记处理 `Weak` 引用 | 循环引用导致内存泄漏 | 在父子关系中使用 `Weak` |
| 过度使用 `Arc<Mutex<T>>` | 性能瓶颈 | 考虑消息传递或原子操作 |

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
- [The Rust Programming Language Book](https://doc.rust-lang.org/book/) — Rust 官方教程
- [Rust Style Guide](https://doc.rust-lang.org/style-guide/) — 官方代码风格
- [Cargo Workspaces](https://doc.rust-lang.org/cargo/reference/workspaces.html) — 工作空间文档
- [Rust Reference](https://doc.rust-lang.org/reference/) — 语言参考

### 社区资源

- [Rust Patterns](https://rust-unofficial.github.io/patterns/) — 设计模式合集
- [Rust Anti-Patterns](https://rust-unofficial.github.io/patterns/anti_patterns/) — 反模式参考
- [Rust for Rustaceans](https://rust-for-rustaceans.com/) — 高级 Rust 编程
- [Rust Atomics and Locks](https://marabos.nl/atomics/) — 并发编程深度指南
- [The Rustonomicon](https://doc.rust-lang.org/nomicon/) — Unsafe Rust 黑魔法书
- [rust-patterns-book](https://github.com/your-org/rust-patterns-book) — 本最佳实践的来源书籍

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

- [[Rust 类型模式]] — 泛型、Trait、Newtype 等类型系统模式
- [[Rust 并发与运行时]] — 异步与并发编程专题
- [[Rust 系统级生产环境]] — 系统级与生产环境专题
- [[后端架构]] — Rust 在后端架构中的应用场景
