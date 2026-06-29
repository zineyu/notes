---
tags:
  - wiki
  - Methodologies
  - page
---
> 来源：[Kent Beck - Canon TDD](https://tidyfirst.substack.com/p/canon-tdd)、[Martin Fowler - TDD](https://martinfowler.com/bliki/TestDrivenDevelopment.html)、[Kent Beck - Test-Driven Development: By Example](https://www.oreilly.com/library/view/test-driven-development/0321146530/)

## 核心原则

1. **红-绿-重构（Red-Green-Refactor）** — 先写失败的测试（红），写最少代码使其通过（绿），然后重构改善设计。这是TDD的核心循环。
2. **测试优先（Test First）** — 在写任何功能代码之前，先写自动化测试。这强制你先思考接口设计，分离接口与实现。
3. **测试列表（Test List）** — 在开始实现前，列出所有需要覆盖的测试场景，然后逐个处理。这避免遗漏边界情况。
4. **小步前进（Baby Steps）** — 一次只写一个测试，一次只做一个小的改动。不要一次性写多个测试或大量实现代码。
5. **消除重复（Eliminate Duplication）** — 重构阶段的目标是消除代码和测试中的重复。重复是设计的信号，提示需要抽象。

## 项目目录/架构规范

```
src/
├── main/
│   └── java/
│       └── com/example/
│           └── money/              # 生产代码
│               ├── Money.java
│               ├── Expression.java
│               └── Bank.java
└── test/
    └── java/
        └── com/example/
            └── money/              # 测试代码（镜像生产代码包结构）
                ├── MoneyTest.java
                └── BankTest.java
```

## 标准代码示例

### 经典的Money示例（Kent Beck）

```java
public class MoneyTest {
    
    @Test
    public void testMultiplication() {
        Money five = Money.dollar(5);
        assertEquals(Money.dollar(10), five.times(2));
        assertEquals(Money.dollar(15), five.times(3));
    }
    
    @Test
    public void testEquality() {
        assertTrue(Money.dollar(5).equals(Money.dollar(5)));
        assertFalse(Money.dollar(5).equals(Money.dollar(6)));
        assertFalse(Money.franc(5).equals(Money.dollar(5)));
    }
    
    @Test
    public void testSimpleAddition() {
        Money five = Money.dollar(5);
        Expression sum = five.plus(five);
        Bank bank = new Bank();
        Money reduced = bank.reduce(sum, "USD");
        assertEquals(Money.dollar(10), reduced);
    }
}
```

```java
public class Money implements Expression {
    protected int amount;
    protected String currency;
    
    public Money(int amount, String currency) {
        this.amount = amount;
        this.currency = currency;
    }
    
    public static Money dollar(int amount) {
        return new Money(amount, "USD");
    }
    
    public Expression times(int multiplier) {
        return new Money(amount * multiplier, currency);
    }
    
    public Expression plus(Expression addend) {
        return new Sum(this, addend);
    }
    
    @Override
    public boolean equals(Object object) {
        Money money = (Money) object;
        return amount == money.amount && currency().equals(money.currency());
    }
}
```

## 常见反模式

| 反模式 | 描述 | 后果 |
|--------|------|------|
| **跳过重构（Skipping Refactor）** | 测试通过后直接写下一个测试，从不重构代码 | 代码逐渐腐化，技术债务累积，最终变成"有测试的烂代码" |
| **虚假通过（False Positive）** | 删除断言或复制计算值到期望中来"让测试通过" | 测试失去验证价值，无法发现真正的bug |
| **一次性写所有测试** | 将测试列表中所有测试一次性写成具体代码 | 当实现第一个测试时发现设计决策需要改变，导致大量返工 |

---

**参考来源**：
- [Kent Beck - Canon TDD](https://tidyfirst.substack.com/p/canon-tdd)
- [Martin Fowler - Test Driven Development](https://martinfowler.com/bliki/TestDrivenDevelopment.html)
- [Kent Beck - Test-Driven Development: By Example](https://www.oreilly.com/library/view/test-driven-development/0321146530/)

---

## 相关笔记

- [[BDD]] — TDD 的扩展，聚焦业务行为
- [[DDD]] — 领域驱动设计与 TDD 的协同
