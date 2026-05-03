---
type: garden
topic: Agent
created: 2026-05-03
updated: 2026-05-03
sources: ["raw/Articles/我是怎么用Codex嗨大了的？.md"]
---

# AI 编码工作流

## 概述
使用 AI Agent（Codex、Claude Code）进行软件开发的完整工序，核心思想是将传统软件工程规范接入 AI 辅助开发流程。

## 核心工序

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
- 使用 Trellis 等 harness 框架编排多轮开发
- 核心：trellis-meta → trellis-brainstorm → trellis-update-spec → 多轮开发
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

## 相关页面
- [[需求驱动开发]] — 100-500 问题表技术详解
- [[Skill 设计]] — Agent Skill 设计方法论

## 演化记录
- [2026-05-03] 从 [[raw/Articles/我是怎么用Codex嗨大了的？.md]] 提取
