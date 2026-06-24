---
type: page
topic: Languages
created: 2026-06-24
updated: 2026-06-24
sources: ["raw/Articles/Rust 可观测性实战：别再 println 了.md"]
---

# Rust 可观测性

## 概述

Rust 生产级可观测性应同时覆盖日志、指标和链路追踪：用 `tracing` 建立结构化上下文，用 Prometheus/OTel 采集指标和 trace，并在异步边界、敏感参数、日志级别和遥测 flush 上设置护栏。

## 核心内容

### 三位一体

可观测性不是单纯“多打日志”，而是三类信号的组合：

- **Logs**：回答“发生了什么”，适合记录离散事件和错误信息。
- **Metrics**：回答“有多严重”，重点看错误率、吞吐、P99/P999 延迟等分布指标。
- **Traces**：回答“一次请求经过了哪里”，适合定位跨服务链路中的慢点和断点。

典型数据流是：日志输出到 stdout 后进入 Loki/ELK/CloudWatch；trace 经 OTLP 进入 OTel Collector、Jaeger 或 Grafana Tempo；指标由 Prometheus scrape 并在 Grafana 中展示。

### `tracing` 优先于 `println!` / `log`

`tracing` 的关键能力是 **Span**：它表示一个可嵌套的时间段，并让内部 event 自动继承上下文。相比平面日志，Span 能避免在每条日志里重复手写 `request_id`、`user_id` 等字段。

常见基础配置：

```rust
use tracing_subscriber::{layer::SubscriberExt, util::SubscriberInitExt, EnvFilter};

fn init_tracing() {
    tracing_subscriber::registry()
        .with(EnvFilter::from_default_env())
        .with(tracing_subscriber::fmt::layer())
        .init();
}
```

`#[instrument]` 适合函数级追踪，但必须显式跳过敏感或巨大的参数：

```rust
#[tracing::instrument(skip(password), fields(user_id = %user))]
async fn login(user: &str, password: &str) -> Result<Token, Error> {
    // password will not be recorded
}
```

### 异步 Span 的核心陷阱

不要在异步代码里手动 `span.enter()` 后跨越 `.await`：guard 与线程绑定，而 Future 可能在不同线程之间被 poll，导致 Span 上下文污染或断裂。

```rust
// ❌ Bug-prone: guard crosses .await
let span = tracing::info_span!("process_request");
let _guard = span.enter();
let result = db.query("SELECT ...").await;

// ✅ Bind span to the future
use tracing::Instrument;

let result = db.query("SELECT ...")
    .instrument(tracing::info_span!("process_request"))
    .await;
```

优先使用 `#[instrument]` 或 `.instrument()`；同时建议启用并提升 `clippy::await_holding_span_guard` 的严格度。

### 生产日志级别与测试日志

- 测试中如果看不到日志，通常是没有初始化 subscriber；使用 `try_init()` 而不是 `init()`，避免并行测试重复初始化 panic。
- 生产发布版可用 `tracing` 的 `max_level_info` 等 feature 在编译期移除 `debug!` / `trace!`，避免日志量、格式化和 Interest 检查带来的额外成本。
- 运行期用 `RUST_LOG` / `EnvFilter` 做局部动态调试，发布版本则尽量用编译期最大级别兜底。

### 指标选择

如果只需要 Prometheus 指标，`metrics` + `metrics-exporter-prometheus` 足够轻量；如果已经使用 OpenTelemetry 做 trace，则指标也走 OTel 管线更一致。

指标命名要从一开始统一，例如：

- `http_requests_total`
- `http_request_duration_seconds`
- `http_errors_total`

三类基础指标：

| 类型 | 用途 | 示例 |
| --- | --- | --- |
| Counter | 只增不减计数 | 请求数、错误数 |
| Gauge | 瞬时值 | 当前连接数、队列长度 |
| Histogram | 分布 | 请求延迟 P50/P99/P999 |

不要只看平均值；P99/P999 更接近尾部用户体验。

### OpenTelemetry 与 OTLP

在 Rust 生态中，日志和指标 API/SDK 更稳定，Traces 仍可能经历 API 调整。实践上应使用 OTLP 导出到 Jaeger、Grafana Tempo 或 OTel Collector；旧的 `opentelemetry-jaeger` 路线不应作为新项目默认选择。

`tracing-opentelemetry` 作为 `tracing::Layer`，可以把已有 Span 转成 OpenTelemetry Span。注意 `opentelemetry-otlp` 默认 feature 可能引入较重的 gRPC 依赖；仅需 HTTP 导出时应关闭 default features 并只开启需要的传输。

### HTTP 服务集成

在 axum/tower 体系里，典型组合是：

- `TraceLayer::new_for_http()`：为每个 HTTP 请求创建 Span。
- `PropagateRequestIdLayer::x_request_id()`：传播 request id。
- `traceparent`：跨服务传递 trace context。
- 进程退出前调用 provider 的 shutdown/flush，避免最后一批遥测数据丢失。

默认不要记录请求体和响应体：这会显著影响性能，并可能泄露敏感数据。必要时只记录经过脱敏的关键字段。

### 错误上下文：SpanTrace

`tracing-error` 的 `SpanTrace` 能在错误被捕获时记录当前 Span 链路，比纯函数调用栈更贴近业务上下文。它适合补足 `anyhow::Context` / `thiserror` 无法表达的“当时请求正在做什么”。

## 实践清单

- [ ] 用 `tracing` / `tracing-subscriber` 替换 `println!`。
- [ ] 给入口 handler、关键业务函数、数据库访问加 `#[instrument]`。
- [ ] 对密码、token、请求体、大对象使用 `skip(...)`。
- [ ] 禁止 `span.enter()` guard 跨 `.await`。
- [ ] 测试中用 `try_init()` 或 `test-log`。
- [ ] 生产编译配置限制最大日志级别，运行期用 `EnvFilter` 做局部调试。
- [ ] 指标采用统一命名，并使用 Histogram 观察尾延迟。
- [ ] 使用 OTLP，而非旧式 Jaeger 专用导出。
- [ ] HTTP 请求启用 TraceLayer 和上下文传播。
- [ ] 优雅关闭时 flush/shutdown 遥测 provider。

## 相关页面

- [[Rust]] — Rust 最佳实践总览
- [[Rust 并发与运行时]] — 异步运行时与 `.await` 边界
- [[Rust 系统级生产环境]] — 错误处理、测试、生产配置
- [[DevOps/运维与 SRE]] — 可观测性在 SRE 中的角色

## 演化记录

- [2026-06-24] 从 [[raw/Articles/Rust 可观测性实战：别再 println 了.md]] 提取
