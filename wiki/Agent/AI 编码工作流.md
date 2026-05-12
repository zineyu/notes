---
type: page
topic: Agent
created: 2026-05-03
updated: 2026-05-12
sources: ["raw/Articles/我是怎么用Codex嗨大了的？.md", "raw/Articles/从VibeCoding到HarnessEngineering再到Trellis落地.pdf"]
---

# AI 编码工作流

## 概述
使用 AI Agent（Codex、Claude Code）进行软件开发的完整工序，核心思想是将传统软件工程规范接入 AI 辅助开发流程：Vibe Coding 推动需求快速进入可验证状态，[[Harness Engineering]] 让流程稳定可控，[[Trellis]] 把规范、任务、上下文、验证和沉淀工程化落地。

## 核心工序

### 0. Vibe Coding 闭环
Vibe Coding 的核心不是“写得快”，而是尽快把想法推进到可讨论、可验证、可迭代的状态。它强调从抽象需求进入可视化和可执行对象：

```text
需求澄清 → PRD → 原型图 → 前端 → 后端 → 测试 → 反向迭代
```

原型图把抽象需求变成可视化讨论对象；PRD / Spec 定义“做到什么程度才算完成”；测试和评测把“看起来能用”变成“稳定能用”，并反向推动规格和实现迭代。

### 1. 项目初始化
- 创建项目文件夹，建立 `docs/` 和 `prompts/` 两个子目录
- `docs/`：UML、架构设计、文档规则、唯一真相源
- `prompts/`：工程阶段控制、目标控制、每个阶段的素材与约束文件

### 2. 上下文管理
- 维护一个统一的 prompt 文件，记录所有项目每个环节的 prompt
- 作用：复用收藏的 prompts、把控项目进度、迭代规范、锻炼系统思考能力
- 定期保存项目快照，防止开发跑偏

### 3. 需求深化（两视角法）
- **开发视角**：清晰边界、制定规范
- **产品视角**：提升可用性
- 生成 100-500 个问题的需求表让用户回答
- 提示词关键：不限制模型泛化能力、允许冗余开发
- 答案确认后解耦为 PRD + Spec

### 4. 多模型流水线
- 先让 opus 4.7 跑一轮，再用 gpt-5.5 补充一轮
- 确保 spec 和 prd 的完善性
- 开发阶段使用 gpt-5.5（自纠错能力强）

### 5. Harness 驱动开发
- 使用 [[Trellis]] 等 harness 框架编排多轮开发
- 核心：把规则、上下文、任务、验证和历史沉淀接入 Agent 生命周期
- 个人项目通常先做好 `AGENTS.md` / `CLAUDE.md`、lint、Spec；长期项目再引入 Trellis 等框架
- 长期任务使用 codex-autoresearch skill

### 6. 检验与结项
- 按 spec/prd 逐项手测
- 注意：模型可能出现"未开发但标记已完成"的幻觉
- 人作为最后兜底

## 工具栈

| 工具                  | 类型      | 用途        |
| ------------------- | ------- | --------- |
| Trellis             | harness | 项目开发全流程编排 |
| codex-autoresearch  | skill   | 长期任务自动运行  |
| sequential-thinking | MCP     | 复杂推理      |
| memory              | MCP     | 记忆管理      |
| serena              | MCP     | 代码库理解     |
| playwright          | MCP     | 浏览器操作     |
| Grok-Search         | MCP     | 搜索        |

## 核心理念
> "AI + coding 无非就是将 AI 的开发流程接入传统 coding 的开发规范中，这是唯一正确的发展规律。"

传统软件工程（系统工程、控制论）是 AI 编码的知识基础。vibe → spec → harness → agent 是有规律可循的。

真正优秀的 AI Coding，不是让模型替人写完代码，而是设计一套系统，使模型持续稳定地把事情做对。

## 相关页面
- [[需求驱动开发]] — 100-500 问题表技术详解
- [[Skill 设计]] — Agent Skill 设计方法论
- [[Harness Engineering]] — 让 AI 编码长期稳定可控的系统层
- [[Trellis]] — Harness Engineering 的工程化落地方案

## 演化记录
- [2026-05-12] 从 [[raw/Articles/从VibeCoding到HarnessEngineering再到Trellis落地.pdf]] 补充 Vibe Coding 闭环、Harness 和 Trellis 落地关系
- [2026-05-03] 从 [[raw/Articles/我是怎么用Codex嗨大了的？.md]] 提取
