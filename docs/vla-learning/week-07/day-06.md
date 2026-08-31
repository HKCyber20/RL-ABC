# 第 7 周 Day 6：未见组合与组合泛化

[填写本日 Notebook 作业](../notebooks/week-07/day-06.ipynb)

[上一课：Day 5](day-05.md) · [本周首页](README.md) · [下一课：Day 7](day-07.md)

## 1. 今日目标与预计时间

预计 120 分钟。执行严格的 seen/unseen-combination 评估，区分词汇、模板、视觉位置和属性组合四类泛化，不用混合平均数掩盖差异。

## 2. 这在 VLA 中的作用

VLA 的价值之一是按语言组合已知概念。若“红”“方块”“蓝”“盒子”都见过，但特定组合未见，成功才初步说明组合泛化；若含新词，则测试的是另一问题。

## 3. 核心概念、指标与切分

评估分组：

```text
ID: seen attributes + seen combinations + seen position range
UC: seen attributes + unseen combinations
UP: seen attributes/combinations + unseen positions
UT: equivalent semantics + unseen surface template
```

每组报告 `successes/n`、首步方向一致率、最远阶段和逐 slot 准确/失败率。泛化差距：`Gap = SR_ID - SR_UC`。

## 4. 分步学习

1. 重新运行 Day 2 集合断言，冻结评估清单与 seeds。
2. 检查 UC 的所有 token/slot 在训练中出现且场景不重复。
3. 对 ID 和 UC 各至少 10 次低资源 rollout；主路径建议各 20 次。
4. 按 src_color、dst_color、object、container、位置区间分组。
5. 计算差距并查看是否由某单一属性或位置偏差驱动。
6. 给失败标注 language binding、visual localization、control、timeout、success checker。

## 5. 必做作业

- 提交 frozen split 清单与四类泄漏检查。
- 输出 ID/UC 分组结果，至少含成功数/总数和阶段指标。
- 选择一个“词都认识但组合失败”的 episode，画出指令、场景、动作和阶段时间线。

## 6. 输入、预期输出与验证

输入：Day 2 split、Day 3/4 checkpoint、固定 ID/UC seeds；低资源各 10 次。

预期输出：得到真实的 `SR_ID`、`SR_UC` 和 Gap；课程不预设 UC 必须高，负结果是诊断依据。

验证：从逐 rollout 记录重新汇总；验证所有 UC token 非 OOV；每个属性在 train 计数大于 0；不同评估组不得复用同一 episode id；复跑至少 2 个 seed。

## 7. 固定提交格式

```markdown
# W7D6 提交
- 实际用时：
## ID/UC 定义与冻结清单
## 四类泄漏检查
## 分组成功率/阶段指标
## 泛化差距与属性分析
## 组合失败时间线
## 自测答案
```

## 8. 评分与验收

- UC 定义满足原子已见：20 分。
- split/词汇/episode/位置审计：20 分。
- ID/UC rollout 有原始记录：25 分。
- 分组与 Gap 分析：20 分。
- 失败时间线、复跑和自测：15 分。

总分 100，80 分通过；UC 含 OOV、评估组 episode 重复、只报混合总成功率或隐去负结果时不通过。

## 9. 常见错误与排查

| 错误 | 排查 |
|---|---|
| UC 恰好也全是新位置 | 分离组合与位置因素，分别建 UC/UP |
| 某属性训练仅出现 1 次 | 报覆盖计数，不把稀疏记忆误称泛化 |
| 成功率样本太少 | 报分子/分母与不确定性，避免过度结论 |
| 模板变化导致 OOV | 先测 canonical template 的 UC |

## 10. 时间不足时的最低完成线

在 CPU Mock 上 ID/UC 各 10 次，完成 token 覆盖、组合无交集、episode 无交集和位置平衡四项检查。

## 11. 可选提高

加入 UP/UT 独立评估；用配对 seeds 比较；为失败做混淆矩阵或 bootstrap 区间。

## 12. 完成后的自测题

1. 怎样形式化“组合没见过但原子见过”？
2. 为什么 UC 与新位置要分开？
3. Gap 很大可能来自哪些非语言因素？
4. 样本少时如何避免过度宣称？

[上一课：Day 5](day-05.md) · [本周首页](README.md) · [下一课：Day 7](day-07.md)
