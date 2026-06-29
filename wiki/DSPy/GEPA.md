---
type: page
topic: DSPy
created: 2026-06-02
updated: 2026-06-02
sources: ["raw/Articles/dspy_gepa_analysis.md"]
tags:
  - DSPy
---
# GEPA：让 LLM 当「进化教练」

## 概述
将 Prompt 优化视为自然语言空间的进化问题。核心创新是**反射变异**（Reflective Mutation）：不是随机改字，而是让 LLM 分析失败案例后针对性重写指令。同时引入 Per-Instance Pareto 前沿和系统感知合并，实现复杂系统的深度调优。

## 核心思路

传统遗传算法 vs GEPA：

| 维度 | 传统遗传算法 | GEPA |
|------|-------------|------|
| 编码 | 二进制/实数向量 | **自然语言文本** |
| 适应度 | 标量分数 | **标量分数 + 文本反馈** |
| 选择 | 轮盘赌、锦标赛 | **Pareto 前沿随机抽样** |
| 变异 | 随机位翻转 | **LLM 基于失败分析的语义重写** |
| 交叉 | 单点/多点拼接 | **LLM 系统感知融合** |
| 评估 | 全种群全评估 | **小批量筛选 + 全量验证** |

## 核心创新

### 1. 反射变异（Reflective Mutation）

传统遗传算法的随机变异在自然语言空间效率极低。GEPA 的反射变异流程：

1. 选一小批训练样本（默认 3 个）让当前候选做
2. 记录完整执行轨迹（trace）
3. 找出失败的 predictor
4. 构建**反射数据集**：输入 + 错误输出 + metric 反馈文本
5. 让 Reflection LLM 诊断问题并生成改进指令
6. 替换目标 predictor 的指令，其他保持不变

> Reflection LLM 需足够强（建议 GPT-5 级别），因为它要理解复杂语义、诊断根因、生成有效改进。

### 2. Per-Instance Pareto 前沿

传统优化只关心整体平均分，GEPA 为**每个验证样本**独立追踪最佳得分：

- 候选 A 在样本 1 上强但样本 2 上弱
- 候选 B 在样本 2 上强但样本 1 上弱
- 传统优化会丢弃 B；GEPA 保留两者在 Pareto 前沿上
- 后续的 **Merge** 可把 A 和 B 的优点组合

### 3. 系统感知合并（Merge）

让 LLM 基于两个候选的**重叠成功案例**生成融合版本：
- 只在有足够重叠验证样本（默认 ≥5 个）时触发
- LLM 分析 "A 擅长 X，B 擅长 Y，怎么融合？"
- 生成融合指令后评估，更好则接受

## 双层评估策略

| 层级 | 样本数 | 目的 | 成本 |
|------|--------|------|------|
| **小批量筛选** | 3 个 | 快速验证反射变异是否有效 | 极低（6 次 metric 调用） |
| **全量验证** | 完整验证集 | 精确评估，更新 Pareto 前沿 | 较高 |

小批量只有 3 个的原因：成本低、多样性高、筛选效率高（在小批量上没改进的候选在大样本上大概率也不行）。

## 关键参数

| 参数 | 作用 | 默认值 | 建议 |
|------|------|--------|------|
| `reflection_lm` | 执行反射变异的 LLM | — | **必须设置**，建议最强模型 |
| `reflection_minibatch_size` | 每次反射的样本数 | 3 | 保持默认 |
| `candidate_selection_strategy` | 选择父候选的策略 | `"pareto"` | pareto 保持多样性，current_best 更激进 |
| `use_merge` | 是否启用合并 | `True` | 建议开启 |
| `auto` | 预算预设 | — | light/medium/heavy 三档 |
| `max_metric_calls` | 最大评估次数（硬预算） | — | 三者选一个设置 |

## 适用场景

最适合的场景：
- **复杂多阶段系统**：ReAct、多轮对话、多 predictor 协作
- **Metric 能提供丰富反馈**：不只是 0/1，还能说出"哪里错了、为什么错"
- **追求极限性能**：愿意消耗更多 API 调用换取更高质量
- **成本敏感但追求效果**：论文显示 GEPA 比强化学习（如 GRPO）少用 **35 倍** rollouts

决策树：
```
任务复杂（多 predictor 流水线）? → 否 → BootstrapFewShot 或 MIPROv2
                                  → 是 → Metric 能提供文本反馈? → 否 → 先改造 metric
                                                              → 是 → 预算充足且有强 Reflection LM? → 否 → MIPROv2
                                                                                                  → 是 → GEPA
```

## 相关页面
- [[BootstrapFewShot]] — GEPA 可与之配合使用（GEPA 优化指令，BootstrapFewShot 提供例题）
- [[MIPROv2]] — 预算有限时的替代方案

## 演化记录
- [2026-06-02] 从 inbox 归档创建
