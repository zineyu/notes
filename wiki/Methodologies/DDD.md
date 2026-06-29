---
tags:
  - wiki
  - Methodologies
  - page
---
> 来源：[Eric Evans - Domain-Driven Design](https://en.wikipedia.org/wiki/Domain-driven_design)、[Microsoft Azure Architecture Center](https://learn.microsoft.com/en-us/azure/architecture/microservices/model/tactical-domain-driven-design)、[ddd-leaven-v2](https://github.com/BottegaIT/ddd-leaven-v2) (880⭐)

## 核心原则

1. **聚焦领域（Focus on the Domain）** — 对于复杂业务系统，领域逻辑应是首要关注点。代码结构（类名、方法名、变量名）应匹配业务领域术语。
2. **统一语言（Ubiquitous Language）** — 开发团队与领域专家使用相同的业务语言交流，该语言应贯穿代码、文档、对话，避免"翻译成本"。
3. **限界上下文（Bounded Context）** — 拒绝单一统一模型。将大型系统划分为多个限界上下文，每个上下文有自己的领域模型和语言。
4. **聚合设计（Aggregate Design）** — 定义一致性边界。每个聚合有且仅有一个根实体，外部只能通过根实体访问内部对象。设计小聚合，仅包含必须在单一事务中保持一致的数据。
5. **领域事件（Domain Events）** — 使用领域事件实现跨聚合/微服务的最终一致性。事件应描述有意义的领域变化（如"订单已取消"），而非技术变化（如"记录已插入"）。

## 项目目录/架构规范

```
src/
├── domain/                      # 领域层（核心）
│   ├── aggregates/             # 聚合根
│   ├── entities/               # 实体
│   ├── value-objects/          # 值对象
│   ├── events/                 # 领域事件
│   ├── services/               # 领域服务
│   └── repositories/           # 仓储接口（仅接口）
├── application/                 # 应用层
│   ├── commands/               # 命令处理器
│   ├── queries/                # 查询处理器
│   └── services/               # 应用服务（编排用例）
├── infrastructure/              # 基础设施层
│   ├── persistence/            # 持久化实现
│   ├── messaging/              # 消息队列
│   └── external/               # 外部服务适配器
└── interfaces/                  # 接口适配层
    ├── web/                    # REST API
    ├── cli/                    # 命令行接口
    └── events/                 # 事件处理器
```

## 标准代码示例

### 聚合根（Aggregate Root）

```java
@Entity
@AggregateRoot
public class Shipment extends BaseAggregateRoot {
    
    private AggregateId orderId;
    private ShippingStatus status;
    
    // 封装的业务行为
    public void ship() {
        if (status != ShippingStatus.WAITING) {
            throw new IllegalStateException("cannot ship in status " + status);
        }
        status = ShippingStatus.SENT;
        eventPublisher.publish(new OrderShippedEvent(orderId, getAggregateId()));
    }
    
    public void deliver() {
        if (status != ShippingStatus.SENT) {
            throw new IllegalStateException("cannot deliver in status " + status);
        }
        status = ShippingStatus.DELIVERED;
        eventPublisher.publish(new ShipmentDeliveredEvent(getAggregateId()));
    }
}
```

### 值对象（Value Object）

```java
@Embeddable
public class Money {
    public static final Money ZERO = new Money(BigDecimal.ZERO, "USD");
    
    private BigDecimal denomination;
    private String currencyCode;
    
    // 值对象是不可变的
    public Money add(Money other) {
        if (!this.currencyCode.equals(other.currencyCode)) {
            throw new IllegalArgumentException("Cannot add different currencies");
        }
        return new Money(this.denomination.add(other.denomination), this.currencyCode);
    }
}
```

## 常见反模式

| 反模式 | 描述 | 后果 |
|--------|------|------|
| **贫血领域模型（Anemic Domain Model）** | 实体只有getter/setter，业务逻辑全部放在Service中 | 领域逻辑泄漏到应用层，模型失去表达能力，难以维护 |
| **大聚合体（Big Aggregates）** | 聚合包含过多对象，试图在一次事务中保持大量数据一致性 | 并发冲突增加、性能下降、事务范围过大 |
| **通用语言不一致** | 代码术语与业务术语不一致，不同上下文使用不同词汇描述同一概念 | 沟通成本增加、需求理解偏差、系统难以演进 |

---

**参考来源**：
- [Eric Evans - Domain-Driven Design](https://en.wikipedia.org/wiki/Domain-driven_design)
- [Microsoft Azure - Tactical DDD](https://learn.microsoft.com/en-us/azure/architecture/microservices/model/tactical-domain-driven-design)
- [ddd-leaven-v2 GitHub](https://github.com/BottegaIT/ddd-leaven-v2)

---

## 相关笔记

- [[TDD]] — 驱动领域模型演进的方法论
- [[BDD]] — 统一语言的业务行为验证
- [[后端架构]] — DDD 在架构设计中的应用
