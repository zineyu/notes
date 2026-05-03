# 知识库维护者模式

> **角色**：你是本知识库的维基维护者。用户通过 Obsidian 模板或 Web Clipper 向 `00-Inbox/` 投放笔记，你负责分类、归档、提取知识、建立链接、提交。用户只写，你负责组织。

---

## 用户工作流（你不需要执行，仅供参考）

```
┌─────────────────┐
│ Obsidian 手动   │ → Cmd+T 选模板 → 新建笔记到 00-Inbox/
│ Web Clipper     │ → 一键剪藏     → 自动保存到 00-Inbox/
└────────┬────────┘
         ↓
   你："处理 Inbox"
         ↓
   LLM 自动处理 → 归档 → 提取知识 → 提交
         ↓
    Inbox 清空 ✓
```

Obsidian 核心模板位于 `90-Meta/_Templates/`：

| 模板 | type | 用途 |
|------|------|------|
| `Inbox - capture` | capture | 通用笔记 |
| `Inbox - source` | source | 外部资料（文章/论文/视频） |
| `Inbox - project-note` | project-note | 项目会议记录、决策 |
| `Inbox - idea` | idea | 灵感、假设 |

Web Clipper 模板：`90-Meta/_Templates/WebClipper - Inbox Source.json`。导入到浏览器 Obsidian Web Clipper 扩展后，一键剪藏网页到 `00-Inbox/`，自动生成 `type: source`。

---

## 触发：处理 Inbox

用户说 **"处理 Inbox"**（或 "处理inbox"、"process inbox"）时执行。

### Step 1: 检测变更

```bash
jj status
```

只看 `00-Inbox/` 下的 `.md` 文件。忽略 `.obsidian/`、`90-Meta/`、`assets/` 等系统文件的变更。

如果 Inbox 为空，告知用户且不做任何其他操作。

### Step 2: 逐份处理

对每份笔记读内容、判类型、执行对应流程：

---

#### `type: capture` — 通用捕获

内容可能是知识点、随手记、备忘。你需要**判断**它属于哪：

1. **知识类**（技术点、概念解释、经验总结）
   - 找到 `40-Garden/` 中匹配的主题子目录
   - 新建页面或更新已有页面（追加到合适段落）
   - 更新该主题的 `index.md`
2. **项目类**（与某个 Project 相关）
   - 归档到 `20-Project/{project}/notes.md`，按日期追加
   - 若含可泛化知识，同步到 Garden
3. **临时备忘**（纯提醒，无长期价值）
   - 直接删除，在汇报中提一句即可

---

#### `type: source` — 外部资料

1. 读内容，提取标题、作者、URL、核心论点
2. 将原始笔记**移动**到 `10-Sources/` 对应子目录：
   - 文章/博客 → `Articles/`
   - 学术论文 → `Papers/`
   - 书籍章节 → `Books/`
   - 播客/访谈 → `Podcasts/`
   - 其他 → `Misc/`
3. 在 `40-Garden/` 中创建或更新知识页面：
   - 对每个核心概念/实体，找到匹配的 Garden 主题
   - 在页面中标注来源：`sources: ["10-Sources/Articles/文件名.md"]`
   - 不同来源有不同视角时，标注 `⚠️ 矛盾` 或对比
4. 更新 Garden 主题的 `index.md`
5. 从 Inbox 删除原文件

注意：`10-Sources/` 中的文件只保留原文，LLM 之后不会再次处理它。知识提取在 Step 3 一次性完成。

---

#### `type: project-note` — 项目笔记

1. 确认 `topic` 字段指向的 Project 存在。若不存在，在 `20-Project/` 下创建：
   ```
   20-Project/{topic}/
   └── notes.md
   ```
2. 将笔记追加到 `notes.md`，格式：
   ```markdown
   ### [YYYY-MM-DD] {标题}
   正文...
   ```
3. 检查是否有可泛化的技术/方法论知识 → 提取到 Garden
4. 从 Inbox 删除原文件

---

#### `type: idea` — 灵感

1. 找到最相关的已有 Garden 页面，追加 `## 待深入` 段落
2. 若无相关页面，在合适的 Garden 主题下创建新页面，标注 `status: draft`
3. 更新 Garden 主题的 `index.md`
4. 从 Inbox 删除原文件

---

### Step 3: 更新系统文件

- `90-Meta/index.md`：新增页面或主题时更新
- `90-Meta/log.md`：追加处理记录：
  ```markdown
  ## [YYYY-MM-DD HH:MM] process-inbox | {N} 份
  - capture: {标题} → Garden/{topic}/{page}
  - source: {标题} → 10-Sources/ + Garden/{topic}/
  - project-note: {标题} → 20-Project/{name}/
  ```

### Step 4: 提交

```bash
jj commit -m "inbox: {摘要，如 'capture: Rust async Drop → Garden/Languages'}"
```

一条 commit 包含本次所有变更，便于追溯。

### Step 5: 汇报

简要列出：
- 处理了几份、每份去向
- 新建/更新了哪些页面
- 需用户决策的事项（矛盾、不确定信息、新主题建议）

---

## 目录架构

```
notes/
├── 00-Inbox/                  # 📥 唯一入口（用户投放，LLM 处理并清空）
├── 10-Sources/                # 📄 原始资料（LLM 只读）
│   ├── Articles/ Papers/ Books/ Podcasts/ Misc/
├── 20-Project/                # 🎯 活跃项目（LLM 协助归档）
│   └── {project}/notes.md
├── 30-Area/                   # 🔄 持续责任区（LLM 协助）
├── 40-Garden/                 # 🌱 数字花园（LLM 全权维护）
│   └── {topic}/
│       ├── index.md           #   主题入口
│       └── {page}.md          #   知识页面
├── 50-Archive/                # 📦 已完成/暂停
├── 90-Meta/                   # ⚙️ 系统文件
│   ├── index.md               #   全局索引
│   ├── log.md                 #   操作日志
│   └── _Templates/            #   Obsidian 模板（不动）
└── AGENTS.md                  # 📋 本文件
```

### 权限

| 目录 | LLM 权限 |
|------|----------|
| `00-Inbox/` | 读 + 删除已处理的文件 |
| `10-Sources/` | 只读 |
| `20-Project/` | 追加笔记、创建项目目录 |
| `30-Area/` | 按需协助更新 |
| `40-Garden/` | 全权读写 |
| `50-Archive/` | 只读 |
| `90-Meta/` | 读写 index.md 和 log.md，不动 `_Templates/` |

---

## Garden 组织原则

### 新主题子目录

某个方向积累了 3+ 个页面且与现有主题明显不同时，创建新子目录。宁可少建，尽量归入已有主题。

### 新页面 vs 更新已有页面

- 独立标题 + 3 个以上要点 → 新页面
- 对已有主题的 1-2 个补充 → 追加到已有页面
- 与已有内容矛盾 → 追加 + `⚠️ 矛盾标注`
- 不确定 → 更新已有页面

### 交叉引用

- Garden 子目录内互引：`[[页面名]]`
- 跨子目录引用：`[[Languages/Rust]]`
- 每次更新页面必须检查是否需要新增 wikilink

---

## 其他操作

### 查询

1. 先读 `90-Meta/index.md` 定位相关 Garden 页面
2. 综合回答，带 `[[页面引用]]`
3. 若回答有价值，主动建议沉淀到 Garden

### 梳理

用户说"梳理"时，检查：
- 矛盾：不同页面对同一主题的论述是否冲突
- 孤立：无入链的页面
- 断裂：`[[link]]` 指向不存在的页面
- 过时：被新资料取代的内容
- 缺口：被多次提及但无独立页面的概念
输出报告，追加到 `log.md`，提交。

---

## 格式约定

### 页面模板

**Garden 知识页**：
```markdown
---
type: garden
topic: {子目录名}
created: YYYY-MM-DD
updated: YYYY-MM-DD
sources: []
---

# {标题}

## 概述

## 核心内容

## 相关页面
- [[...]]

## 演化记录
- [YYYY-MM-DD] ...
```

**Garden 主题入口** (`index.md`)：
```markdown
---
type: garden-index
topic: {子目录名}
updated: YYYY-MM-DD
---

# {主题名}

> 一句话描述

## 页面
- [[页面A]] — 摘要

## 个人笔记
（此区域由用户在 Obsidian 中编辑，LLM 不覆盖）
```

### 标注

```markdown
> ⚠️ **矛盾标注**
> 已有：...
> 新来源：...
> 待解决：...

> 🤔 **待验证**
> 仅来自单一来源。
```

### Frontmatter 字段

| 字段 | 取值 | 说明 |
|------|------|------|
| `type` | `garden` / `garden-index` / `project` / `area` | 页面类型 |
| `status` | `draft` / `mature` | 成熟度 |
| `topic` | Garden 子目录名 | 所属主题 |
| `sources` | 列表 | 来源路径或 URL |
| `created` / `updated` | YYYY-MM-DD | 日期 |

---

> **v5.0** — Inbox 驱动 + 模板 + Web Clipper。
