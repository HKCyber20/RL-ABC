# 第 7 周 Day 2：指令设计、组合覆盖与数据切分

[填写本日 Notebook 作业](../notebooks/week-07/day-02.ipynb)

[上一课：Day 1](day-01.md) · [本周首页](README.md) · [下一课：Day 3](day-03.md)

## 1. 今日目标与预计时间

预计 90–120 分钟。定义可解析的指令 grammar，审计颜色、物体、容器和模板覆盖，并构建不泄漏的 seen/unseen-combination split。

## 2. 这在 VLA 中的作用

语言泛化取决于数据组合。如果某种颜色总出现在固定位置或某个模板只出现在测试集，模型可能利用伪相关或遭遇未知表达，而不是学习语言 grounding。

## 3. 核心概念与数据结构

结构化指令：

```text
把 {src_color} {src_object} 放进 {dst_color} {dst_container}
```

一条语义记录应同时保存：`surface_text` 与 `slots={verb,src_color,src_object,dst_color,dst_container}`。未见组合要求每个原子属性在训练集中出现，但特定笛卡尔组合被留出：

\[
C_{test}\cap C_{train}=\varnothing,\quad
Atoms(C_{test})\subseteq Atoms(C_{train}).
\]

## 4. 分步学习

1. 列出任务支持的源/目标属性及同义表达，冻结 canonical slot 值。
2. 写生成器从 slot 生成文本与 metadata，避免仅靠字符串反解析。
3. 生成组合覆盖矩阵，统计每个组合的 episode 数、成功数和位置分布。
4. 选择至少 2 个 held-out 组合；确保其中每个属性在 train 出现。
5. 按 episode 分割并去重，检查同一场景 seed 或近邻帧不跨 split。
6. 检查指令长度、模板和目标位置是否与标签形成伪相关。

## 5. 必做作业

- 提交 grammar、slot 字典和至少 12 条文本—slot 对照。
- 输出组合覆盖矩阵与 split 清单；低资源路径至少 4 个 train 组合、2 个 held-out 组合。
- 做 5 个审计：属性覆盖、组合交集、episode 交集、重复文本、目标位置偏差。

## 6. 输入、预期输出与验证

输入：第 5 周轨迹或 synthetic 双目标 episode；颜色/物体/容器至少各两个可区分值。

预期输出：held-out 组合不在 train，但其单个 slot 值都在 train；split 无 episode 重叠；每条文本有唯一 canonical slots。均为预期结果。

验证：用集合断言；随机抽 10 条文本与场景目标人工核对；交换 surface template 但保持 slots，确认标签任务不变；报告无法平衡的偏差。

## 7. 固定提交格式

```markdown
# W7D2 提交
- 实际用时：
## Grammar 与 canonical slots
## 12 条文本/slot 对照
## 组合覆盖矩阵
## Train/val/test 切分规则
## 五项泄漏/偏差审计
## 自测答案
```

## 8. 评分与验收

- grammar 和 metadata 可逆/可审计：20 分。
- 组合覆盖矩阵：20 分。
- held-out 定义满足原子已见：20 分。
- episode 去重与五项审计：25 分。
- 人工抽查与自测：15 分。

总分 100，80 分通过；把 OOV 指令误称未见组合、同一 episode 跨 split 或文本与场景不一致时不通过。

## 9. 常见错误与排查

| 错误 | 排查 |
|---|---|
| test 含训练未见词 | 先做 vocabulary coverage，再谈组合泛化 |
| 同一 seed 换模板跨 split | 按 scene/episode group 切分，不按文本行 |
| 颜色固定对应位置 | 统计每个颜色的目标位置分布并随机化 |
| 文本解析歧义 | 直接保存 slots，surface 只作输入 |

## 10. 时间不足时的最低完成线

建立 2×2 颜色—目标组合、4 train + 2 held-out 任务，完成属性覆盖、组合交集、episode 交集三个断言。

## 11. 可选提高

加入两种表面模板；生成 paraphrase split；用互信息或简单分类器检测位置—词语伪相关。

## 12. 完成后的自测题

1. 未见词、未见模板与未见组合为何要分开？
2. 为什么 surface_text 和 slots 都应保存？
3. scene seed 泄漏如何高估语言泛化？
4. 颜色固定位置会让模型学到什么捷径？

[上一课：Day 1](day-01.md) · [本周首页](README.md) · [下一课：Day 3](day-03.md)
