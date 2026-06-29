---
type: page
topic: DSPy
created: 2026-06-02
updated: 2026-06-02
sources: ["raw/Articles/dspy_bootstrap_fewshot_analysis.md"]
tags:
  - DSPy
---
# BootstrapFewShot：让好老师帮你挑例题

## 概述
用一个更强的模型（Teacher）在训练集上做一遍，把做对的题的解题过程保存为 few-shot 示例（demo），交给学生模型（Student）使用。全自动、带质量过滤的例题收集器。

## 核心流程

1. **准备 Teacher**：可以是更大模型、更高温度或更多上下文
2. **遍历训练集**：逐题让 Teacher 做，metric 判断对错
3. **保存做对的 trace**：记录完整输入→输出路径，作为 demo
4. **注入 Student**：将收集的 demo 填入 Student predictor 的 prompt

### 关键设计

| 机制 | 说明 |
|------|------|
| **Leave-one-out** | Teacher 做某题时临时将其从 demos 中移除，防止背答案 |
| **Trace 捕获** | 自动记录多步 pipeline 中每个 predictor 的输入输出 |
| **多轮尝试** | 第一次做错可换参数重试，提高收集成功率 |
| **去缓存** | 更换 `rollout_id` 和温度，确保每次尝试获得不同输出 |

## 家族变体

| 变体 | 核心差异 | 适用场景 |
|------|---------|---------|
| **基础版** | 顺序收集前 N 道对的题 | 快速 baseline，数据量小 |
| **RandomSearch** | 生成多种 demo 配置，验证集打分选最优 | 搜索最佳 demo 数量和顺序 |
| **KNNFewShot** | 运行时按输入相似度动态检索例题 | 输入差异大，需个性化 |
| **Optuna** | Bootstrap 生成大量候选，贝叶斯优化选最佳组合 | demo 多，需精细筛选 |
| **Finetune** | 将 trace 变成微调数据训练底层模型 | 追求极限性能 |

## 参数速查

| 参数 | 作用 | 默认 | 注意 |
|------|------|------|------|
| `metric` | 判断 Teacher 是否做对 | — | 必须提供 |
| `teacher_settings` | Teacher 的特殊配置 | `{}` | 想让 Teacher 更强时设置 |
| `max_bootstrapped_demos` | 最多收集多少自举例题 | `4` | 数据多可增大 |
| `max_labeled_demos` | Student 看到的**所有例题总数**上限（自举+原始） | `16` | 名字有误导性，不是仅原始标注 |
| `max_rounds` | 每道题最多尝试几次 | `1` | 数据难可提高 |

## 适用决策树

```
有可靠 metric? → 否 → 先定义 metric
                → 是 → 有比 Student 强的 Teacher? → 否 → 考虑 MIPROv2/GEPA
                                                → 是 → 数据量?
                                                    <50  → BootstrapFewShot（快速 baseline）
                                                    50-200 → BootstrapFewShotWithRandomSearch
                                                    >200  → 考虑 MIPROv2（联合优化指令+例题）
```

## 优缺点

**优点**
- 全自动：提供 metric 和数据集即可
- 质量过滤：只有 Teacher 做对的才被采纳
- 支持复杂流水线：多 predictor 各自独立获取相关例题
- 扩展性强：基础组件被 RandomSearch、Optuna、Finetune 复用

**缺点**
- 依赖 Teacher 质量：Teacher 不够强 → 成功率低 → 例题少甚至为空
- 顺序截断：按训练集顺序收集，先遇到的优先
- 静态例题：编译后固定（KNNFewShot 除外）
- Metric 敏感：设计不当导致偏差

## 相关页面
- [[MIPROv2]] — 数据量大时的联合优化方案
- [[GEPA]] — 复杂系统深度调优的进化算法方案
- [[Agent/SKILL 设计指南]] — DSPy 的 Skill/Module 设计思路可与此类比

## 演化记录
- [2026-06-02] 从 inbox 归档创建
