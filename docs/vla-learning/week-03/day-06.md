# 第 3 周 Day 6：Rollout 评估、成功率与失败分类

[填写本日 Notebook 作业](../notebooks/week-03/day-06.ipynb)

[← 上日：协变量偏移](day-05.md) · [周首页](README.md) · [次日：周项目 →](day-07.md)

## 1. 今日目标与预计时间

预计 100–130 分钟。建立可复现 rollout 协议，报告成功率、最终距离、步数与失败类型；理解 denominator、seed、终止条件和置信不确定性。

## 2. 今日知识点及其在 VLA 中的作用

VLA 的最终指标是任务是否完成。成功率必须绑定明确条件、最大步数和测试分布。失败分类将“感知、语言绑定、规划、控制、超时、接口错误”拆开；低维阶段先用到达、超时、越界、数值异常等类别练习。

## 3. 必要概念、公式、形状与数据流

N 次 rollout 成功率：

\[
\hat p=\frac{\sum_i \mathbf{1}[success_i]}{N}
\]

同时报告：`N`、seed/初态列表、horizon、控制频率、成功阈值、mean/median final distance、time-to-success、越界数。每次 episode 一行原始记录：

```text
episode_id, seed, split, success, steps,
final_distance, max_error, termination_reason, failure_type
```

成功率样本少时波动大；今天至少 20 次，后续项目会提高。

## 4. 分步骤学习

1. 写不可在结果出现后更改的评估协议。
2. 固定 20 个 seed/初态并保存。
3. 运行 policy 与 expert/random 两个基线。
4. 从原始记录聚合指标，核对分母。
5. 为每次失败赋一个主类型并摘录证据。

## 5. 必做作业

对 Day 5 环境执行至少 20 次 rollout，比较 imitation policy 与 random/zero action 基线。成功条件、horizon 在运行前写明。提交逐 episode 记录、汇总表和至少 3 个失败案例（若失败不足 3 个，可通过困难初态单独做诊断集，不混入正式成功率）。说明离线 validation loss 与成功率是否排序一致。

## 6. 输入、预期输出与验证方法

- 输入：冻结的 policy checkpoint/参数、固定 20 seed、评估配置。
- 预期：每策略 20 行记录，汇总计数与原始行一致；具体成功率不预设。
- 验证：重新聚合 `sum(success)/N`；成功 episode 的 final distance 满足阈值；termination reason 枚举覆盖所有行；相同 seed 用于公平比较。

低资源方案：使用解析策略、电子表格聚合，不需训练神经网络。

## 7. 提交内容与固定格式

固定包含：`冻结协议`、`策略/基线版本`、`20次原始记录`、`汇总指标`、`失败案例`、`loss与成功率比较`、`复跑命令/步骤`、`投入分钟数`。

## 8. 评分与验收标准

100 分，80 分通过：协议 20；原始记录 20；指标正确 20；基线公平 15；失败分类 15；离线/闭环分析 10。无分母、事后改阈值、丢弃失败 seed、只报最好一次均为关键失败。

## 9. 常见错误与排查

- 超时 episode 不计入分母：所有启动的正式 rollout 都应记录。
- checkpoint 在评估中继续训练：设 eval/freeze 并记录版本。
- 初态集合不同：用同 seed 列表。
- 平均 final distance 被少数异常值主导：同时给 median 和失败明细。

## 10. 时间不足时的最低完成线

冻结协议并执行 20 次轻量数值 rollout；至少提交成功率、最终距离和终止原因。

## 11. 可选提高任务

用 bootstrap 或 Wilson 区间给成功率不确定性；明确方法、重复次数/公式，不把区间当额外成功证据。

## 12. 完成后的自测题

1. 报告 90% 成功率但不说 N 有何问题？
2. 为什么要保存逐 episode 记录？
3. 失败类型和 termination reason 有何区别？
4. validation loss 更低的模型为何可能成功率更差？

[← 上日：协变量偏移](day-05.md) · [周首页](README.md) · [次日：周项目 →](day-07.md)
