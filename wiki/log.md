# 操作日志

> LLM Wiki 按时间顺序的操作记录。每行以固定前缀开头，可用 `grep` 解析。

## [2026-05-10] migrate | LLM Wiki 架构重构
- 目录重构：`10-Sources/` → `raw/`，`40-Garden/` → `wiki/`
- 模板移至 `Templates/`（根目录），便于访问
- 全局索引和日志移至 `wiki/index.md` 和 `wiki/log.md`（由 LLM 维护）
- AGENTS.md 重写为 LLM Wiki 模式（ingest / query / lint）
- 移除 PARA 空目录（00-Inbox, 20-Project, 30-Area, 50-Archive, 90-Meta）

## [2026-05-03] process-inbox | 1 份
- source: 我是怎么用Codex嗨大了的？ → 创建 [[Agent/AI 编码工作流]]、[[Agent/需求驱动开发]]，更新 [[Agent/Skill 设计]]、[[Agent/index]]

## [2026-05-03] refactor v3 | PARA 为主体，Garden 为知识核心
- 架构重构为完整 PARA：20-Project、30-Area、40-Garden、50-Archive
- Garden 取代原 Wiki：知识在 PARA 的 Resource 层，按主题分子目录
- 20-Wiki/Concepts 内容分散到 40-Garden/ 各主题目录
- 30-MOC 内容融入各 Garden 子目录的 index.md
- 元文件移至 90-Meta
- 24 份概念页面作为 Garden 的种子内容
- 6 个 Garden 子目录：Frontend, Languages, Architecture, DevOps, Methodologies, Agent

## [2026-05-03] refactor v2 | 知识系统与行动系统分离

## [2026-05-03] init | 初始架构
