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

## Reference: Python script

通用批量更新脚本，可按项目调整 `WIKI_DIR`、`SOURCE_TAG_MAP`、`KEYWORD_PATTERNS`。

```python
#!/usr/bin/env python3
"""通用 wiki tag 批量更新脚本。前置：pip install pyyaml"""

from pathlib import Path
import re
import yaml

WIKI_DIR = Path("wiki")

# (正则, tag)。长/具体模式写在前，避免短模式误匹配
KEYWORD_PATTERNS = [
    (r"\bRust\b", "Rust"),
    (r"\bGo\b", "Go"),
    (r"\bKubernetes\b", "Kubernetes"),
    (r"\bK8s\b", "Kubernetes"),
    (r"\betcd\b", "etcd"),
    (r"\bNix\b", "Nix"),
    (r"Home\s*Manager", "Home-Manager"),
    (r"\bRedis\b", "Redis"),
    (r"\bSQL\b", "SQL"),
    (r"\bTypeScript\b", "TypeScript"),
    (r"\bJavaScript\b", "JavaScript"),
    (r"\bReact\b", "React"),
    (r"Tailwind\s*CSS", "Tailwind"),
    (r"\bCSS\b", "CSS"),
    (r"\bHTML5?\b", "HTML"),
    (r"\bVite\b", "Vite"),
    (r"\bDDD\b", "DDD"),
    (r"\bTDD\b", "TDD"),
    (r"\bBDD\b", "BDD"),
    (r"\bDSPy\b", "DSPy"),
    (r"BootstrapFewShot", "BootstrapFewShot"),
    (r"MIPROv2", "MIPROv2"),
    (r"GEPA", "GEPA"),
    (r"\bCodex\b", "Codex"),
    (r"\bClaude\b", "Claude"),
    (r"Harness\s*Engineering", "Harness-Engineering"),
    (r"\bTrellis\b", "Trellis"),
    (r"Sub-agent", "sub-agent"),
    (r"\bSRE\b", "SRE"),
    (r"\bAI\b", "AI"),
    (r"\bSkill\b", "Skill"),
]

SOURCE_TAG_MAP = {
    "raw/Articles/": "article",
    "raw/Papers/": "paper",
    "raw/Books/": "book",
    "raw/Podcasts/": "podcast",
    "raw/Misc/": "misc",
}


def parse_frontmatter(text):
    m = re.match(r"^(---\s*\n)(.*?)\n(---\s*\n?)", text, re.DOTALL)
    if not m:
        return None, text
    return m.group(2), text[m.end():]


def infer_base_tags(path: Path):
    rel = path.relative_to(WIKI_DIR)
    parts = rel.parts
    if rel.name == "index.md":
        topic = parts[0] if len(parts) > 1 else "wiki"
        tags = ["wiki", topic, "index"]
    elif rel.name == "log.md":
        tags = ["wiki", "log"]
    else:
        topic = parts[0] if parts else "wiki"
        tags = ["wiki", topic, "page"]
    seen = set()
    return [t for t in tags if not (t in seen or seen.add(t))]


def source_type_tags(sources):
    if not sources:
        return []
    tags = []
    for s in sources:
        for prefix, tag in SOURCE_TAG_MAP.items():
            if s.startswith(prefix) and tag not in tags:
                tags.append(tag)
    return tags


def content_keywords(title, filename):
    text = f"{title} {filename}"
    found = []
    for pat, tag in KEYWORD_PATTERNS:
        if re.search(pat, text, re.IGNORECASE) and tag not in found:
            found.append(tag)
    return found


def extract_existing_tags(fm_text):
    tags = []
    in_tags = False
    for line in fm_text.splitlines():
        if re.match(r"^tags:", line):
            in_tags = True
            continue
        if in_tags:
            if re.match(r"^[A-Za-z]", line) and not re.match(r"^\s*- ", line):
                break
            m = re.match(r"^\s*- (.*)", line)
            if m:
                t = m.group(1).strip()
                if t and t not in tags:
                    tags.append(t)
    return tags


def set_tags_fm(fm_text, tags):
    lines = fm_text.splitlines()
    out = []
    i = 0
    tags_present = False
    while i < len(lines):
        line = lines[i]
        if re.match(r"^tags:", line):
            tags_present = True
            out.append("tags:")
            i += 1
            while i < len(lines):
                l = lines[i]
                if re.match(r"^\s*- ", l):
                    i += 1
                    continue
                if l.strip() == "":
                    j = i
                    while j < len(lines) and lines[j].strip() == "":
                        j += 1
                    if j < len(lines) and re.match(r"^[A-Za-z]", lines[j]) and not re.match(r"^\s*- ", lines[j]):
                        break
                    i += 1
                    continue
                break
            for t in tags:
                out.append(f"  - {t}")
            continue
        out.append(line)
        i += 1
    if not tags_present:
        out.append("tags:")
        for t in tags:
            out.append(f"  - {t}")
    result = "\n".join(out)
    if not result.endswith("\n"):
        result += "\n"
    return result


def run():
    files = sorted(p for p in WIKI_DIR.rglob("*.md") if p.is_file())
    for f in files:
        text = f.read_text(encoding="utf-8")
        fm, body = parse_frontmatter(text)
        base = infer_base_tags(f)
        title = ""
        sources = []
        if fm:
            try:
                fm_dict = yaml.safe_load(fm) or {}
                title = str(fm_dict.get("title") or "")
                sources = fm_dict.get("sources") or []
            except Exception:
                pass
        final = list(base)
        for t in source_type_tags(sources) + content_keywords(title, f.stem) + extract_existing_tags(fm or ""):
            if t not in final:
                final.append(t)
        if fm is None:
            new_text = "---\n" + set_tags_fm("", final) + "---\n\n" + text
        else:
            new_fm = set_tags_fm(fm, final)
            new_text = "---\n" + new_fm + "---\n" + body
        if new_text != text:
            f.write_text(new_text, encoding="utf-8")
            print("updated", f)


if __name__ == "__main__":
    run()
```

## Lessons learned

- tag 不能含空格，否则 Obsidian 解析异常。
- 处理已有 `tags:` 时，必须给返回值补换行再拼接 closing `---`。
- 提取已有 tags 时只读取 `tags:` 块内的列表项，不要误把 `author:` 等其他 list 字段当作 tag。
- 目录结构是主题 tag 的最稳定来源，但不要止步于此；`wiki` + `page`/`index`/`log` 等元 tag 能支撑更灵活的查询和过滤。
- 把加 tag 作为 process-inbox / ingest 的默认收尾步骤，可避免 wiki 页面长期缺失标签。
- 内容关键词匹配要注意顺序与词边界，防止 `AI` 被 `Tailwind` 误触发。

## Checklist

- [ ] 已列出全部目标文件。
- [ ] 每个文件都包含 `wiki` + 主题 + 类型三层基础 tag。
- [ ] 已有自定义 tag 被保留。
- [ ] 没有含空格的 tag。
- [ ] 抽样文件 frontmatter 解析正常。
- [ ] diff 检查通过并已提交。
