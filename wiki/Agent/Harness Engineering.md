---
type: page
topic: Agent
created: 2026-05-12
updated: 2026-06-24
sources: ["raw/Articles/从VibeCoding到HarnessEngineering再到Trellis落地.pdf"]
tags:
  - wiki
  - Agent
  - page
  - article
  - Harness-Engineering
---
# Harness Engineering

## 概述
Harness Engineering 是围绕 AI Agent 的系统化工程约束：不只优化单次提示词或上下文，而是设计防偏、控制、验证、修复和沉淀机制，让模型能力在真实项目中持续稳定地产出。

## 核心内容

### 三层嵌套关系

AI 编码实践可以分成三层，它们不是替代关系，而是逐层扩展：

1. **Prompt Engineering**：优化人类给模型的指令文本，解决“怎么写指令”。
2. **Context Engineering**：设计模型看到的完整输入，包括工具定义、RAG 结果、消息历史、输出模式和 MCP，解决“给模型看什么”。
3. **Harness Engineering**：围绕 Agent / 工作流设计完整系统，包括 linter、CI、任务拆分、权限、可观测性、自动清理和文档保鲜，解决“防止什么、控制什么、修复什么”。

可以把 Agent 理解为：

```text
Agent = Model + Harness
```

模型负责生成能力本身，Harness 负责把能力导向项目目标，并让过程可控、可验证、可持续。

### 典型失败模式

没有 Harness 时，模型可能单次表现很强，但在长任务和真实代码库中会暴露系统性问题：

- 不稳定遵守规范
- 记不住任务进度
- 不知道自己哪里错了
- 跨会话后质量退化
- 产出与 PRD / Spec 脱节

Harness 的价值不是让模型“更聪明”，而是把开发过程变成一个有护栏、有反馈、有沉淀的工程系统。

### 常见实现形态

当前个人开发者可选的 Harness 方案大致分三类：

- **纯配置方案**：通过 `CLAUDE.md`、`AGENTS.md`、skills、lint 规则和项目文档，把约束层手动落地。
- **框架增强方案**：如 [[Trellis]]、ccg-workflow，在配置层上增加 Spec 系统和 hook 自动注入，使规范持续进入每次交互。
- **多角色编排方案**：如 oh-my-claudecode、oh-my-opencode，通过 planner、executor、reviewer、debugger 等角色分工提高复杂任务推进能力；具体设计时要区分可复用的工具型 Agent 与依赖共享上下文的 [[Sub-agent 与工具型 Agent|sub-agent]]。

### 成本与退化风险

Harness 本身也会产生维护成本。每个组件都编码了一个假设：模型自己不能完成某件事。随着模型能力提升，这些假设需要持续压力测试。

如果大量时间消耗在维护计划文件、进度文档、Agent 通信和工作流本身，而实际代码产出很少，说明 Harness 已经从助力变成负担。好的 Harness 应该让约束自动化、轻量化，并且随模型能力提升而可删除。

## 相关页面
- [[AI 编码工作流]] — Harness 是 AI 编码闭环中让过程稳定可控的系统层
- [[Trellis]] — Trellis 是 Harness Engineering 的工程化落地方案
- [[Skill 设计]] — Skill 是 Harness 中可复用的上下文和行为单元
- [[Sub-agent 与工具型 Agent]] — 说明多 Agent Harness 中工具边界与委派边界的差异

## 演化记录
- [2026-05-12] 从 [[raw/Articles/从VibeCoding到HarnessEngineering再到Trellis落地.pdf]] 提取
- [2026-06-24] 补充工具型 Agent 与 sub-agent 的编排边界
