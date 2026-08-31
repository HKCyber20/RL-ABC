# 第 7 周 Day 7：语言条件策略周项目验收

[填写本日 Notebook 作业](../notebooks/week-07/day-07.ipynb)

[上一课：Day 6](day-06.md) · [本周首页](README.md) · [下一课：第 8 周 Day 1](../week-08/day-01.md)

## 1. 今日目标与预计时间

预计 2–3 小时。冻结并验收一个 `image + instruction + state → action` 策略，以 baseline、反事实、seen/UC rollout 和失败分类形成完整证据链。

## 2. 这在 VLA 中的作用

这是最小语言条件控制闭环。通过后，第 8 周将围绕动作表示、训练流水线和 action chunk 把各部件整合成最小 VLA，而不再改变输入语义。

## 3. 核心概念与验收链

```text
数据审计
→ tokenizer/text encoder 可复现
→ 三模态 forward/backward
→ paired instruction probe
→ autonomous rollout
→ ID/UC 分组与失败归因
```

对照至少包含“正确语言策略”和“固定指令/语言置零”之一。若 cross-attention 未优于 concat，也应以实际结果选择更简单模型。

## 4. 分步学习

1. 冻结 tokenizer、split、normalization、模型配置与 checkpoint 标识。
2. 复跑一个固定 batch，验证重载输出一致。
3. 在同一场景切换两条冲突指令，记录动作/目标选择。
4. 运行 ID 与 UC 各至少 10 次低资源或 20 次主路径 autonomous rollout。
5. 与无语言 baseline 或正确文本打乱结果对照。
6. 分类至少 3 个失败案例；闭卷完成周测。
7. 写第 8 周输入接口：image/state/token_ids/mask 及 shape。

## 5. 必做作业

- 提交可重载 checkpoint、词表、config、split 和 normalization 标识。
- 提交 paired probe、语言消融、ID/UC 原始逐 rollout 记录和汇总。
- 提交至少 3 个失败案例，其中语言绑定类若不存在须解释筛选证据，不能虚构。

## 6. 输入、预期输出与验证

输入：本周最佳但预先冻结的 checkpoint、固定评估 seeds、同图冲突指令和 UC 清单。

预期输出：策略对语义变化有可测响应并形成真实成功率；不预设结果必须达到某个百分比。若失败，需能定位主要瓶颈并给出最小补救实验。

验证：新进程重载；success 从逐 episode 记录重算；随机交换文本时其余输入逐元素相同；评估无专家纠偏；所有结果标注实际运行设备与耗时。

## 7. 固定提交格式

```markdown
# W7D7 周项目提交
- 实际用时与设备：
## 冻结资产清单
## 重载一致性证据
## Paired instruction 与语言消融
## ID/UC rollout 表（成功数/总数）
## 三个失败案例与最小补救
## 周测答案
## 第 8 周输入接口
```

## 8. 评分与验收

- 冻结资产完整可重载：15 分。
- 同图换指令的行为证据：20 分。
- 语言消融对照：15 分。
- ID/UC autonomous rollout：25 分。
- 失败分类与补救：15 分。
- 周测与接口冻结：10 分。

总分 100，80 分通过；文本/场景错配、仅展示 attention、无逐 rollout 记录或评估含专家纠偏时不通过。

## 9. 常见错误与排查

| 错误 | 排查 |
|---|---|
| checkpoint 重载后文本行为变了 | 核对词表 id、规范化、mask、config 和 eval 模式 |
| paired probe 对但 rollout 失败 | 查动作尺度、闭环频率、视觉遮挡与误差累积 |
| ID 高、UC 低 | 查组合覆盖、属性偏差与 text encoder 组合能力 |
| 无语言 baseline 也成功 | 任务/状态可能泄漏目标，重做受控双目标场景 |

## 10. 时间不足时的最低完成线

CPU synthetic 双目标任务：ID/UC 各 10 次、同图换指令 8 对、语言打乱对照、checkpoint 重载和 1 个失败案例。

## 11. 可选提高

比较 concat/cross-attention；加入等价模板评估；用更严格的随机化减少位置捷径。

## 12. 完成后的自测题

1. 你有哪些行为证据证明语言被使用？
2. 为什么“被使用”仍不等于“正确 grounding”？
3. cross-attention 未胜出时为何可能保留 concat？
4. UC 失败最小补救实验应只改变哪个因素？
5. 第 8 周哪些接口必须保持不变？

[上一课：Day 6](day-06.md) · [本周首页](README.md) · [下一课：第 8 周 Day 1](../week-08/day-01.md)
