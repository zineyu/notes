---
tags:
  - wiki
  - Methodologies
  - page
  - BDD
---
> 来源：[Cucumber官方文档 - BDD](https://cucumber.io/docs/bdd/)、[Cucumber - Better Gherkin](https://cucumber.io/docs/bdd/better-gherkin)、[Cucumber - Anti-patterns](https://cucumber.io/docs/guides/anti-patterns/)

## 核心原则

1. **发现-制定-自动化（Discovery-Formulation-Automation）** — 通过协作工作坊发现需求（Discovery），将示例制定为可执行的规范（Formulation），自动化这些规范作为测试（Automation）。
2. **跨角色协作** — BDD要求业务人员、开发人员和测试人员共同参与需求讨论，建立对问题的共同理解。
3. **具体示例驱动** — 使用具体的、真实的业务示例来说明系统行为，而非抽象的规格说明。
4. **声明式而非命令式** — 场景描述"做什么"（what），而非"怎么做"（how）。避免在场景中出现UI交互细节。
5. **活文档（Living Documentation）** — 可执行的场景同时作为自动化测试和系统文档，始终与实际代码保持同步。

## 项目目录/架构规范

```
src/
├── test/
│   └── resources/
│       └── features/             # Gherkin特性文件
│           ├── user_management/
│           │   ├── registration.feature
│           │   └── login.feature
│           └── order_processing/
│               ├── checkout.feature
│               └── payment.feature
│   └── java/
│       └── stepdefinitions/      # Step定义
│           ├── UserSteps.java
│           └── OrderSteps.java
└── main/
    └── java/
        └── domain/               # 领域代码
```

## 标准代码示例

### Gherkin场景（声明式风格）

```gherkin
Feature: Subscription plans
  In order to read premium content
  As a reader
  I want to be able to subscribe to a plan

  Rule: Free users can read free articles
    Example: Free user reads a free article
      Given a free user
      When they view a free article
      Then they should see the full article

  Rule: Paid users can read all articles
    Example: Subscriber reads a premium article
      Given a user with a premium subscription
      When they view a premium article
      Then they should see the full article

    Example: Free user sees paywall for premium article
      Given a free user
      When they view a premium article
      Then they should see a paywall
```

### Step定义实现（Java）

```java
public class SubscriptionSteps {
    
    @Given("a user with a premium subscription")
    public void aUserWithPremiumSubscription() {
        user = new User();
        user.subscribeTo(Plan.PREMIUM);
    }
    
    @When("they view a premium article")
    public void theyViewPremiumArticle() {
        result = articleService.viewArticle(user, premiumArticle);
    }
    
    @Then("they should see the full article")
    public void theyShouldSeeFullArticle() {
        assertTrue(result.isFullContentVisible());
    }
}
```

## 常见反模式

| 反模式 | 描述 | 后果 |
|--------|------|------|
| **Feature-coupled Step Definitions** | Step定义按特性文件组织（如`edit_work_experience_steps.java`），而非按领域概念组织 | Step定义无法跨特性复用，导致代码爆炸、重复和维护成本高 |
| **连接词步骤（Conjunction Steps）** | 一个步骤包含多个动作（如`Given I have shades and a brand new Mustang`） | 步骤过于专门化，难以复用。应拆分为原子步骤并使用`And`连接 |
| **命令式场景（Imperative Scenarios）** | 场景描述UI交互细节（点击按钮、填写字段） | 场景与UI实现紧耦合，UI变更导致测试大面积失败，难以作为文档阅读 |

---

**参考来源**：
- [Cucumber Official Documentation - BDD](https://cucumber.io/docs/bdd/)
- [Cucumber - Anti-patterns](https://cucumber.io/docs/guides/anti-patterns/)
- [Cucumber - Better Gherkin](https://cucumber.io/docs/bdd/better-gherkin)

---

## 相关笔记

- [[TDD]] — BDD 的基础方法论
- [[DDD]] — 统一语言与 BDD 的协同
