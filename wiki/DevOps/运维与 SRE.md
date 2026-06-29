---
tags:
  - wiki
  - DevOps
  - page
---
> 来源：[Google SRE Book](https://sre.google/sre-book/table-of-contents/)、[Google SRE - Service Level Objectives](https://sre.google/sre-book/service-level-objectives/)、[Google SRE - Eliminating Toil](https://sre.google/sre-book/eliminating-toil/)

## 核心原则

1. **SLO 驱动运维（SLO-Driven Operations）** — 以 Service Level Objective 为核心指标，而非单纯追求 100% 可用性。定义明确的 SLI（如延迟、错误率、吞吐量），并设置错误预算（Error Budget）。
2. **消除 Toil（Eliminate Toil）** — 将手动、重复性、可自动化的运维工作（Toil）控制在 50% 以下。优先投入工程工作来减少未来的 Toil。
3. **拥抱风险（Embrace Risk）** — 100% 可用性既不现实也不必要。允许可控的故障注入和计划内停机，以验证系统弹性和团队响应能力。
4. **可观测性三位一体（Observability）** — Metrics（指标）、Logging（日志）、Tracing（追踪）三位一体。基于症状的告警优于基于原因的告警。
5. **blameless 事后分析（Blameless Postmortems）** — 故障发生后进行无责复盘，聚焦系统性改进而非追究个人责任。

补充到 Rust 服务时，可参考 [[Languages/Rust 可观测性]]：用 `tracing` 组织结构化日志和 Span，用 Histogram 关注 P99/P999，用 OTLP 接入 Jaeger/Grafana Tempo，并在优雅关闭时 flush 遥测数据。

## 项目目录/架构规范

```
sre/
├── slos/                      # SLO 定义
│   ├── api-gateway-slo.yml
│   └── payment-service-slo.yml
├── runbooks/                  # 运维手册
│   ├── on-call/
│   ├── incident-response/
│   └── postmortems/
├── monitoring/
│   ├── prometheus-rules/
│   ├── grafana-dashboards/
│   └── alertmanager/
├── automation/
│   ├── terraform/
│   ├── ansible/
│   └── scripts/
└── docs/
    ├── architecture/
    └── playbooks/
```

## 标准代码示例

### SLO 配置模板

```yaml
# api-gateway-slo.yml
service: api-gateway
slos:
  - name: availability
    display_name: "API Gateway Availability"
    slis:
      - indicator: "successful_requests / total_requests"
        good_events: "http_requests_total{status!~'5..',job='api-gateway'}"
        total_events: "http_requests_total{job='api-gateway'}"
    target: 0.9995  # 3.5 nines
    window: 30d
    
  - name: latency
    display_name: "API Gateway Latency"
    slis:
      - indicator: "histogram_quantile(0.99, rate(http_request_duration_seconds_bucket[5m]))"
    target: 0.200  # 200ms p99
    window: 30d
    alerting:
      burn_rate: 2
      fast_burn_window: 1h
      slow_burn_window: 3d
```

### Prometheus 告警规则（基于 SLO）

```yaml
groups:
- name: slo-alerts
  rules:
  - alert: APIGatewayErrorBudgetBurn
    expr: |
      (
        sum(rate(http_requests_total{job="api-gateway",status=~"5.."}[1h]))
        /
        sum(rate(http_requests_total{job="api-gateway"}[1h]))
      ) > (1 - 0.9995) * 14.4
    for: 2m
    labels:
      severity: critical
      team: platform
    annotations:
      summary: "API Gateway is burning error budget too fast"
      
  - alert: HighLatency
    expr: |
      histogram_quantile(0.99, 
        sum(rate(http_request_duration_seconds_bucket{job="api-gateway"}[5m])) by (le)
      ) > 0.2
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "API Gateway p99 latency > 200ms"
```

## 常见反模式

| 反模式 | 描述 | 后果 |
|--------|------|------|
| **基于阈值而非 SLO 的告警** | 单纯设置 "CPU > 80%" 告警会导致告警疲劳 | 应基于用户可感知的症状（如错误率、延迟）设置 SLO 驱动的告警 |
| **100% 可用性目标** | 追求 100% 可用性会阻碍创新、增加成本，且无法验证恢复能力 | 应设定现实的 SLO 并预留错误预算 |
| **事后追责文化** | 追究个人责任的复盘会导致信息隐藏和恐惧文化 | 应建立 blameless 文化，聚焦系统性改进 |

---

**参考来源**：
- [Google SRE Book](https://sre.google/sre-book/table-of-contents/)
- [Google SRE - Service Level Objectives](https://sre.google/sre-book/service-level-objectives/)
- [Google SRE - Eliminating Toil](https://sre.google/sre-book/eliminating-toil/)
- [Google SRE - Embracing Risk](https://sre.google/sre-book/embracing-risk/)

---

## 相关笔记

- [[Kubernetes]] — 容器编排系统的运维核心
- [[etcd]] — 分布式系统的关键数据存储运维
- [[Languages/Rust 可观测性]] — Rust 服务的日志、指标与链路追踪实践
