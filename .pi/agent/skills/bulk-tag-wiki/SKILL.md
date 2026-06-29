---
name: bulk-tag-wiki
description: Use when the user asks to add, update, or normalize frontmatter tags for all files under a project's wiki/ directory. Applies topic-based tags and preserves existing tags.
author: zine
created: 2026-06-29
updated: 2026-06-29
---

# Bulk Tag Wiki

批量为 LLM Wiki / Obsidian garden 的 Markdown 文件增加或修正 frontmatter `tags`。

## Quick start

1. 确认范围：目标 `wiki/` 路径、tag 规则。
2. 读取项目约定的 frontmatter 格式（如 `AGENTS.md` 中的页面模板）。
3. 列出所有 `wiki/**/*.md`。
4. 按目录结构推断 tag，保留已有 tag。
5. 插入或追加 `tags:` 到 frontmatter。
6. 抽样检查、查看 diff、提交。

## When to use

- 用户说「给 wiki 文件加 tag」、「add tags to wiki」、「批量更新 frontmatter tags」。
- 需要一次性为整组笔记补全分类 tag，而不是逐个手动编辑。

## Workflow

### 1. 推断 tag

规则按路径确定：

| 文件 | tag |
| --- | --- |
| `wiki/{topic}/{page}.md` | `{topic}` |
| `wiki/{topic}/index.md` | `{topic}`, `index` |
| `wiki/index.md` | `wiki`, `index` |
| `wiki/log.md` | `wiki`, `log` |

- tag 使用目录名，避免空格。
- 若用户指定额外 tag，追加到规则之后。

### 2. 写入 frontmatter

- 已有 frontmatter：在结束 `---` 前追加 `tags:` 块；若已存在 `tags:`，则合并缺失项。
- 无 frontmatter：在文件开头插入最小 `---\ntags:\n  - ...\n---\n\n`。
- 不修改其他 frontmatter 字段的顺序和格式。

### 3. 保留与校验

- 保留已有 tag（如 `clippings`），不要覆盖。
- 确保 `tags` 列表以换行结束后再写 `---`，防止 `tag---` 粘连。
- 抽样读取 3-5 个文件确认 frontmatter 正确。

### 4. 记录与提交

- 在 `wiki/log.md` 追加 maintenance 记录。
- 仅提交受影响文件；不要把 `.obsidian/` 等无关改动带进来。

## Lessons learned

- tag 不能含空格，否则 Obsidian 解析异常。
- 处理已有 `tags:` 时，必须给返回值补换行再拼接 closing `---`。
- 目录结构是 tag 的最稳定来源，不要从标题推断（标题可能含空格和特殊字符）。

## Checklist

- [ ] 已列出全部目标文件。
- [ ] tag 规则与目录结构一致。
- [ ] 已有 tag 被保留。
- [ ] 没有含空格的 tag。
- [ ] 抽样文件 frontmatter 解析正常。
- [ ] diff 检查通过并已提交。
