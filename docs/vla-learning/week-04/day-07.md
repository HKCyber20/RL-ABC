# 第 4 周 Day 7：周项目——单步与 Action Chunk 对比

[填写本日 Notebook 作业](../notebooks/week-04/day-07.ipynb)

[← 上日：频率与延迟](day-06.md) · [周首页](README.md) · [次日：第 5 周视觉仿真 →](../week-05/day-01.md)

## 1. 今日目标与预计时间

预计 3–4 小时。整合前四周能力，在同一低维环境、数据划分和 test seeds 上比较单步策略与 action chunk 策略，提交时间对齐、mask、训练、闭环指标和失败复盘。

## 2. 今日知识点及其在 VLA 中的作用

这是进入视觉仿真前的时序基线验收。有效对比不要求 chunk 胜出，而要求：标签未错位、尾部和历史 padding 被正确 mask、模型输出与 scheduler 一致、控制频率和延迟透明、结果可从原始 rollout 重算。

## 3. 必要概念、公式、形状与数据流

公平对比建议：

```text
共同数据/split/normalization/expert
baseline: history H_o -> one action [B,1,d_a]
chunk:    history H_o -> H_a actions [B,H_a,d_a]
共同 environment/control Hz/test seeds/success criterion
chunk 额外报告 replanning interval k
```

训练 loss 对所有有效动作位置求 masked mean；执行日志追踪动作来源。报告 success、final distance、steps、smoothness、latency/age 与 failure type。

## 4. 分步骤学习

1. 闭卷完成周测并修正概念缺口。
2. 审计对齐、history/chunk 索引和 masks。
3. 训练或复用单步模型，训练 chunk MLP/Transformer 小模型。
4. 在同一 20+ test seeds 评估，至少一种扰动/延迟设置。
5. 从逐 episode 记录重算汇总，挑选代表失败而非只挑最好视频。
6. 写明下一周加入图像后哪些接口保持不变。

## 5. 必做作业

周测（25 分）：回答一帧错位、历史窗口边界、padding/causal mask、chunk valid、H_a/k/frequency/latency 的关系。

周项目（75 分）：比较单步与 `H_a>=3` 的 chunk 策略；至少 30 条训练 episode、按 episode split、同一归一化；test 至少 20 seeds；报告正常与一种延迟/扰动设置。chunk 模型可为 MLP，Transformer 至少需完成 Day 5 的 forward/backward 验证，保证低资源可完成。

## 6. 输入、预期输出与验证方法

- 输入：冻结数据 manifest、统计、两模型配置、同一 test seeds、控制协议。
- 预期：两策略均有 checkpoint/参数、训练日志、20+ 原始 rollout、汇总和失败分类；不预期 chunk 必然更优。
- 验证：动力学残差检测时间对齐；padding/chunk 反事实；split 交集为空；汇总可从原始记录复算；动作 source 日志与 scheduler 匹配。

## 7. 提交内容与固定格式

固定包含：`周测`、`数据/时间契约`、`模型与mask`、`训练配置/曲线`、`执行调度`、`20+ seeds原始记录`、`正常/扰动汇总`、`失败案例`、`公平性审计`、`复跑命令`、`总投入时间`。

## 8. 评分与验收标准

100 分，80 分通过：周测 25；时间/数据/mask 20；模型训练 15；调度与频率 15；rollout 对比 20；复现/复盘 5。四个门槛：时间对齐、padding mask、chunk 不跨 episode、执行频率明确。选择性丢弃结果或伪造性能不通过。

## 9. 常见错误与排查

- 单步与 chunk 使用不同训练量：报告并尽量控制数据/更新步数。
- chunk 输出正确但执行反归一化维错：对 `[B,H_a,d_a]` 最后一维处理。
- 尾部 mask 只用于 loss、不用于指标：训练 loss mask 与 rollout 无关，评估只执行可用动作。
- 扰动设置对一个策略更晚/更弱：固定世界时间和幅度。
- 从 loss 宣布 chunk 更好：以闭环指标和失败证据为准。

## 10. 时间不足时的最低完成线

用 MLP 分别输出 1 步和 3 步，在同一 20 seeds 下完成正常 rollout；提交所有门槛证据。延迟消融未完成则登记补救，整周暂不最终通过。

## 11. 可选提高任务

加入 temporal ensemble 或不同 `H_a={2,4,8}` 消融，报告计算量、规划率、平滑度和恢复能力，而非只报最好配置。

## 12. 完成后的自测题

1. 哪些证据说明性能差异不是时间错位造成？
2. 如果 chunk loss 更低但成功率更低，应先检查哪三项？
3. 图像加入后哪些张量和时间契约会变化，哪些保持不变？
4. 你能否从失败 episode 追溯到输入窗口、预测 chunk 和实际执行 offset？

[← 上日：频率与延迟](day-06.md) · [周首页](README.md) · [次日：第 5 周视觉仿真 →](../week-05/day-01.md)
