---
type: page
topic: Agent
created: 2026-05-12
updated: 2026-05-12
sources: ["raw/Articles/从VibeCoding到HarnessEngineering再到Trellis落地.pdf"]
tags:
  - Agent
---
# Trellis

## 概述
Trellis 是面向 AI 编码助手的脚手架框架：通过 hook、spec、任务、验证和会话沉淀，把 [[Harness Engineering]] 工程化、自动化、可持续化。

## 核心内容

### 定位

Trellis 不是单纯的提示词工具，也不只是技能集合，而是把以下要素串成系统：

- 规范
- 任务
- 上下文
- 验证
- 历史沉淀

与其他方案对比：

| 方案 | 定位 |
|------|------|
| Superpowers | 技能包，强调自动触发的 spec、计划和 subagent-driven-development 流程 |
| oh-my-openagent / 多角色套件 | 多角色编排，强调 planner、executor、reviewer、debugger 等分工 |
| Trellis | AI 编码脚手架，强调规范自动注入和工作流工程化 |

### Hook 机制

Trellis 与 Claude Code 的 hook 机制契合，核心是把规范在关键生命周期自动注入，而不是依赖人手动提醒。

三个关键时机：

1. **session-start**：会话启动时注入开发者身份、工作流指南、会话历史、提交记录和活跃任务。
2. **inject-subagent-context**：调用子 Agent 时按角色注入精确上下文，例如实现任务注入 `implement.jsonl`、`prd.md`、`info.md`，检查任务注入 `check.jsonl` 和 `prd.md`。
3. **ralph-loop**：检查完成后自动触发质量验证，把验证变成工作流的一部分。

### 日常使用流

首次使用大致是安装并初始化：

```bash
npm install -g @mindfoldhq/trellis@latest
trellis init --claude -u your-name
```

日常使用可以简化为：

```text
/trellis:start
输入需求并进行 vibe coding
/finish-work
/record-session
```

Bug 修复后，可以通过 `/break-loop` 分析原因，并把原因沉淀到 `.trellis/spec/` 中，避免同类问题反复出现。

### Spec 结构

Trellis 通过 `.trellis/spec/` 把规范沉淀为明确目录。典型结构包括：

- `frontend/`：组件、Hook、状态管理、类型安全、质量、目录结构
- `backend/`：数据库、错误处理、日志、安全质量、目录结构
- `guides/`：跨层思维、代码复用等思维指南

这种结构把临时经验转成可复用规范，再通过 hook 注入到后续会话中。

### 团队协作

Trellis 中每个人可以拥有独立的 `workspace/{name}/` 和 `.developer`，并通过 worktree 物理隔离并行任务。容易冲突的部分是 `spec/` 和 `tasks/`，需要通过 PR 审查、明确 assignee、重要规范变更同步等方式协调。

## 相关页面
- [[Harness Engineering]] — Trellis 是 Harness Engineering 的框架增强方案
- [[AI 编码工作流]] — Trellis 落在从需求到验证闭环的工程化阶段
- [[需求驱动开发]] — Trellis 的 PRD / Spec 机制依赖前期需求澄清质量

## 演化记录
- [2026-05-12] 从 [[raw/Articles/从VibeCoding到HarnessEngineering再到Trellis落地.pdf]] 提取
