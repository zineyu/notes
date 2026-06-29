---
tags:
  - wiki
  - log
---
# 操作日志

> LLM Wiki 按时间顺序的操作记录。每行以固定前缀开头，可用 `grep` 解析。


## [2026-06-29] maintenance | 为所有 wiki 文件增加 tags
- 为 43 个页面添加 frontmatter tags，并确立为 wiki 维护默认步骤
- tag 不再局限于主题目录名，每个页面至少包含三层：`wiki`、主题目录名、`page`/`index`/`log`
- 保留已有自定义 tag（如 `clippings`）
- 更新: [[wiki/index.md]]、[[wiki/log.md]]、[[AGENTS.md]] 及所有主题页面

## [2026-06-29] maintenance | 补充 wiki 内容 tag 与来源类型 tag
- 为 34 个页面追加内容关键词 tag（如 `Rust`、`Kubernetes`、`observability`、`Harness-Engineering`）
- 根据 `sources` 路径追加来源类型 tag（`article`）
- 保留基础三层 tag 与已有自定义 tag
- 更新: [[wiki/log.md]]、[[.agents/skills/bulk-tag-wiki/SKILL.md]] 及 34 个主题页面

## [2026-06-29] maintenance | 将批量加 tag 脚本沉淀进 skill
- 把实际使用的 Python 脚本抽象为通用版本，写入 `.agents/skills/bulk-tag-wiki/SKILL.md`
- 脚本支持基础 tag、来源类型 tag、内容关键词 tag、保留已有自定义 tag
- 验证脚本对当前 wiki 幂等（无文件变更）
- 更新: [[.agents/skills/bulk-tag-wiki/SKILL.md]]、[[wiki/log.md]]

## [2026-06-24] process-inbox | Rust 可观测性与 Agent 编排模式
- 新建: [[Languages/Rust 可观测性]]、[[Agent/Sub-agent 与工具型 Agent]]
- 更新: [[Languages/Rust]]、[[Languages/Rust 系统级生产环境]]、[[Languages/index]]、[[DevOps/运维与 SRE]]、[[Agent/Harness Engineering]]、[[Agent/index]]、[[wiki/index.md]]
- 归档: [[raw/Articles/Rust 可观测性实战：别再 println 了.md]]、[[raw/Articles/Where to use sub-agents versus agents as tools.md]]

## [2026-06-24] process-inbox | Nix Home Manager 最佳实践
- 新建: [[DevOps/Nix Home Manager]]
- 更新: [[DevOps/index]]、[[wiki/index.md]]
- 归档: [[raw/Articles/Nix Home Manager 最佳实践.md]]
## [2026-05-12] process-inbox | 从 Vibe Coding 到 Harness Engineering，再到 Trellis 落地
- 新建: [[Agent/Harness Engineering]]、[[Agent/Trellis]]
- 更新: [[Agent/AI 编码工作流]]、[[Agent/index]]、[[wiki/index.md]]
- 归档: [[raw/Articles/从VibeCoding到HarnessEngineering再到Trellis落地.pdf]]

## [2026-06-02] process-inbox | DSPy 优化器三篇
- 新建主题: [[DSPy/index]]
- 新建: [[DSPy/BootstrapFewShot]]、[[DSPy/MIPROv2]]、[[DSPy/GEPA]]
- 更新: [[wiki/index.md]]
- 归档: [[raw/Articles/dspy_bootstrap_fewshot_analysis.md]]、[[raw/Articles/dspy_gepa_analysis.md]]、[[raw/Articles/dspy_miprov2_analysis.md]]

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
