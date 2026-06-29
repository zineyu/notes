---
created: 2025-04-27
tags:
  - wiki
  - Agent
  - page
  - Skill
---
## [[【SKILL 设计的最佳实践】什么是SKILL？怎么快速写一个优秀的SKILL？#What’s SKILL|What's SKILL]]
SKILL设计的出发点就在于使用**渐进式披露**的方式，使用尽可能少的上下文实现功能的感知，真正实现有效加载上下文！
因此一个优秀的SKILL.md应当同时具备:

> - **较短的概览实现SKILL的调用**
> - **明确的细节解释**来规划工作流
> - 补充**合适的技术细节完成最后的具体实现**

Claude官网对于Skill.md的创作提供了一个最佳实践 [Skill authoring best practices - Claude Docs](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices)
## [[【SKILL 设计的最佳实践】什么是SKILL？怎么快速写一个优秀的SKILL？#SKILL设计的核心原则 | SKILL设计的核心原则]]

SKILL文档主要分为4种组件:

> - 元信息（YAML元数据）： 包括name以及decription唯一标识符，启动时默认加载，需要保证简洁避免冗余
> - 主体内容（Body）： 相当于目录，提供概述、入门指导和资源引用，按需加载所需资源，减少上下文注入
> - 参考文件(Reference File)：详细指南 API参考 以及使用示例，当主体明确链接到参考文件时实现动态加载
> - 脚本文件（Scription）： 提供可执行脚本用于获取输入输出，保证一致性执行

### 一、 命名是起点 （Naming is First Step）

启动SKILL时只会预加载所有SKILL的**名称（name）** 和 **描述（description）**，因此设计合适精炼的名称以及描述时成功调用SKILL的第一步。

SKILL的命名需要清晰地描述技能所提供的活动或能力，使用**动词ing+对象**的命名，避免模糊或者通用描述，导致模型紊乱

> - 正例：processing-pdfs； analyzing-spreadsheets； managing-databases
> - 反例：helper； documents； utils

### 二、. 描述即路标 （Description Is the Signal）

**Description** 用于查找技能，应包括技能的功能和使用时机。

务必具体，并包含关键术语 。不仅要**说明技能的功能**，还要说明**何时使用该技能的具体触发条件/ 上下文** 。

由于大多数情况下 Claude 根据description来实现工具的调用，因此SKILL的description必须做到有区分，使得Claude知道何时选择该技能。避免模糊表述或者冗余

正例：

```yaml
description: Extract text and tables from PDF files, fill forms, merge documents. Use when working with PDF files or when the user mentions PDFs, forms, or document extraction.s
```

反例（模糊表述）：

```yaml
description: Process pdf files.
```

反例（冗余表述）：

```yaml
description: This skill is designed to help users work with PDF documents by providing comprehensive functionality for extracting text content and tabular data from PDF files, as well as filling out form fields in PDF documents, and merging multiple PDF documents together into a single file. It can be used whenever you need to process PDF files, or when users explicitly mention that they want to work with PDFs, or when they talk about forms, or when document extraction is needed, or in any situation where PDF manipulation might be helpful or beneficial to the task at hand.
```

### 三、 简洁是关键（Concise is Key）

这一部分将以目录的方式组织SKILL更具体的参考资源，执行逻辑以及上下文加载,由于主体部分在加载后会直接参与Claude的上下文中，与其他内容存在竞争关系，因此在主体部分中 – “简洁是关键”，这里Claude 官方提出一个基本假设：

**“Claude已经足够聪明，不需要补充常识，只需要补充Claude不了解的背景信息。”**

正例：

```css
## Extract PDF text

Use pdfplumber for text extraction:

\`\`\`python

import pdfplumber

with pdfplumber.open("file.pdf") as pdf:

text = pdf.pages[0].extract_text()
```
```python
反例：

\`\`\`makdown

## Extract PDF text

PDF (Portable Document Format) files are a common file format that contains

text, images, and other content. To extract text from a PDF, you'll need to

use a library. There are many libraries available for PDF processing, but we

recommend pdfplumber because it's easy to use and handles most cases well.

First, you'll need to install it using pip. Then you can use the code below...
```

**特别的，官方建议正文部分不要超过500行**

### 四、 自由度要匹配（Match the Degrees of Freedom）

由于Claude本身具有很高的自由度，有的时候这种自由度可以极大的提升模型的创造能力，但对于部分特殊任务，我们希望通过严格的流程或者具有唯一确定的解决办法，又或者是我们需要满足特定约束，因此我们需要在SKILL中对于不同操作赋予不同自由度，根据自由度的区别主要可以使用下面的方式来进行规范以及约束：

| 自由度等级 | 任务特征 | 指令设计建议 | 典型示例 |
| --- | --- | --- | --- |
| **High** | \- 多种方法都可能成功   \- 决策高度依赖具体上下文   \- 可以通过启发式而非硬规则指导 | \- 给出总体目标和方向性指导   \- 使用原则、检查清单或启发式约束   \- 信任模型根据上下文选择最佳路径 | 代码审查（最佳方式取决于上下文） |
| **Mid** | \- 存在推荐的执行模式   \- 允许一定范围内的变体   \- 行为会受到配置或参数影响 | \- 固定整体结构或流程   \- 通过参数或配置开放可变部分   \- 平衡可控性与灵活性 | 使用参数控制行为的自动化脚本 |
| **Low** | \- 只有一种安全或正确的执行路径   \- 错误代价高，容错空间极小   \- 执行顺序或步骤错误会造成严重后果 | \- 提供明确的护栏和强约束   \- 给出精确步骤和固定执行顺序   \- 避免让模型自行推断关键决策 | 必须按顺序执行的数据库迁移 |

下面给出官方文档中 对于三种不同自由度的示例

### 高自由度（使用文本描述）：

```markdown
## Code review process

1. Analyze the code structure and organization

2. Check for potential bugs or edge cases

3. Suggest improvements for readability and maintainability

4. Verify adherence to project conventions
```

### 中自由度(使用伪代码或带参数的脚本)

```markdown
## Generate report

Use this template and customize as needed:

\`\`\`python

def generate_report(data, format="markdown", include_charts=True):

# Process data

# Generate output in specified format

# Optionally include visualizations
```

### 低自由度(特定脚本+少参数+文本限制)

```markdown
## Database migration

Run exactly this script:

scripts/migrate.py --verify --backup

Do not modify the command or add additional flags.
```

---

## 实践补充：上下文管理

来自 AI 编码实战经验：

- **统一 prompt 文件**：为所有项目维护一个集中的 prompt 文件，记录每个环节的提示词
- **prompts 文件夹**：按日期/阶段组织，装载素材、约束文件、要求
- **效果**：不用打开特定对话翻历史、可迭代规范、锻炼系统思考能力
- **项目快照**：定期保存上下文快照，确认开发不跑偏

详见 [[AI 编码工作流]]。

## 相关页面

- [[SKILL 设计指南]] — 更详细的 SKILL 设计方法论与最佳实践
- [[AI 编码工作流]] — 完整 AI 编码工序
