---
name: bulk-tag-wiki
description: Use when the user asks to add, update, or normalize frontmatter tags for wiki/ files. Tagging is the default final step of wiki maintenance, not limited to the topic directory name.
author: zine
created: 2026-06-29
updated: 2026-06-29
tags: [skill, wiki, maintenance]
---

# Bulk Tag Wiki

批量为 LLM Wiki / Obsidian garden 的 Markdown 文件增加或修正 frontmatter `tags`。

> Tags are keywords or topics that help you quickly find the notes you want.


## Default behavior

给 wiki 文件加 tag 是 wiki 维护的**默认步骤**。每个 wiki 页面都必须带有 `tags` frontmatter，且 tag 不应只局限于主题目录名。

基础 tag 至少包含三层：

1. **`wiki`** — 标识文件属于 wiki 知识库。
2. **主题 tag** — 页面所在主题目录名，如 `Agent`、`Languages`、`DevOps`。
3. **类型 tag** — `page` / `index` / `log`。

在此之上可追加：用户指定的 tag、来源类型 tag、内容关键词 tag、已有自定义 tag。

## 可选 tag 维度

1. **来源类型**：根据 `sources` 路径前缀自动添加：
   - `raw/Articles/` → `article`
   - `raw/Papers/` → `paper`
   - `raw/Books/` → `book`
   - `raw/Podcasts/` → `podcast`
   - `raw/Misc/` → `misc`

2. **内容关键词**：从 frontmatter `title` 或文件名提取核心术语，例如 `Rust`、`Kubernetes`、`observability`、`type-patterns`、`Harness-Engineering`。使用词边界匹配，避免把 `AI` 误识别为 `Tailwind` 的子串。

## When to use

- 用户说「给 wiki 文件加 tag」、「add tags to wiki」、「批量更新 frontmatter tags」。
- 执行 `process-inbox` / `ingest` / 重构后，作为默认收尾步骤确保 tag 完整。
- 用户指出 tag 不应只局限于目录名时，补充 `wiki`、`page`/`index`/`log` 等元 tag。

## Workflow

### 1. 列出目标文件

```bash
wiki/**/*.md
```

### 2. 推断基础 tags

| 文件 | 基础 tags |
| --- | --- |
| `wiki/{topic}/{page}.md` | `wiki`, `{topic}`, `page` |
| `wiki/{topic}/index.md` | `wiki`, `{topic}`, `index` |
| `wiki/index.md` | `wiki`, `index` |
| `wiki/log.md` | `wiki`, `log` |

- tag 使用目录名/类型名，避免空格。
- 若用户指定额外 tag，追加到基础 tags 之后。

### 3. 合并已有 tags

- 读取现有 frontmatter 中的 `tags:`。
- 保留不在基础规则中的自定义 tag（如 `clippings`）。
- 不重复添加已存在的 tag。

### 4. 写入 frontmatter

- 已有 frontmatter：替换或追加 `tags:` 块，保持其他字段顺序和格式不变。
- 无 frontmatter：在文件开头插入最小 `---\ntags:\n  - ...\n---\n\n`。
- 确保 `tags` 列表以换行结束后再写 closing `---`，防止 `tag---` 粘连。

### 5. 校验与提交

- 抽样读取 3-5 个文件确认 frontmatter 正确。
- 查看 diff，确认仅 wiki 文件被修改。
- 在 `wiki/log.md` 追加 maintenance 记录。
- 提交；不要把 `.obsidian/` 等无关改动带进来。

## Lessons learned

- tag 不能含空格，否则 Obsidian 解析异常。
- 处理已有 `tags:` 时，必须给返回值补换行再拼接 closing `---`。
- 提取已有 tags 时只读取 `tags:` 块内的列表项，不要误把 `author:` 等其他 list 字段当作 tag。
- 目录结构是主题 tag 的最稳定来源，但不要止步于此；`wiki` + `page`/`index`/`log` 等元 tag 能支撑更灵活的查询和过滤。
- 把加 tag 作为 process-inbox / ingest 的默认收尾步骤，可避免 wiki 页面长期缺失标签。

## Checklist

- [ ] 已列出全部目标文件。
- [ ] 每个文件都包含 `wiki` + 主题 + 类型三层基础 tag。
- [ ] 已有自定义 tag 被保留。
- [ ] 没有含空格的 tag。
- [ ] 抽样文件 frontmatter 解析正常。
- [ ] diff 检查通过并已提交。
