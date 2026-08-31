# 第 4 周 Day 4：Action Chunk 标签与执行策略

[填写本日 Notebook 作业](../notebooks/week-04/day-04.ipynb)

[← 上日：Sequence Dataset](day-03.md) · [周首页](README.md) · [次日：Transformer Policy →](day-05.md)

## 1. 今日目标与预计时间

预计 100–140 分钟。为每个决策时刻构造未来 `H_a` 步动作标签和 valid mask，理解 chunk 的非自回归/序列输出形式，并实现不同执行/重规划策略。

## 2. 今日知识点及其在 VLA 中的作用

单步策略每次只预测 `a_t`，容易受推理抖动影响；action chunk 一次预测 `[a_t,...,a_{t+H_a-1}]`，可表达短期一致动作并降低高层推理频率。但长 chunk 会更开环，环境变化后修正变慢。训练标签、尾部 mask 和执行调度共同决定效果。

## 3. 必要概念、公式、形状与数据流

对动作序列 `a [T,d_a]`，时刻 t 的标签：

\[
Y_t=[a_t,a_{t+1},\ldots,a_{t+H_a-1}]
\]

形状 `[H_a,d_a]`；超过 episode 末尾的位置 padding，`valid_action [H_a]` 标出真实标签。batch 输出 `[B,H_a,d_a]`。

执行方案：

- open-loop：预测一次，执行完整 chunk；
- receding horizon：每 k 步重新预测，仅执行前 k 步；
- temporal ensemble：重叠 chunk 对同一时刻的预测加权融合。

本周项目采用前两者，避免过早引入复杂融合。

## 4. 分步骤学习

1. 对 `T=5,H_a=3` 手列每个 t 的 chunk 和 valid。
2. 实现 `get_action_chunk(actions,t,H_a)`，禁止跨 episode。
3. 写 masked chunk MSE。
4. 实现执行前 k 步再重规划的 scheduler。
5. 用突发扰动比较 `k=H_a` 与 `k=1` 的恢复。

## 5. 必做作业

使用动作 `[0,1,2,3,4]`（每个可视作 1 维）构造 `H_a=3` 的所有 5 个标签和 valid mask；实现 batch 版本并断言尾部。再用第 3 周环境设计 `H_a=4` 的 perfect/noisy chunk policy，比较 execute-all 与每步重规划，在第 2 步注入位置扰动。提交轨迹和解释，结果以实际实现为准。

## 6. 输入、预期输出与验证方法

- 输入：长度 5 动作、`H_a=3`；轻量到达环境、`H_a=4`、固定扰动。
- 预期：标签 `[5,3,1]`、valid `[5,3]`，有效数 `3+3+3+2+1=12`；chunk 不读取下一 episode。
- 验证：修改 padding 标签不改变 masked loss；scheduler 执行动作可追溯到预测版本与 chunk offset；执行步数和环境步数一致。

低资源方案：手列表格并用规则策略在纸上执行 6 步。

## 7. 提交内容与固定格式

固定包含：`chunk定义`、`T=5标签/mask表`、`构造函数/断言`、`scheduler规则`、`扰动轨迹对比`、`开环-重规划取舍`、`投入分钟数`。

## 8. 评分与验收标准

100 分，80 分通过：标签 25；边界/mask 20；loss 15；scheduler 20；扰动分析 20。跨 episode、尾部 padding 进入 loss、声称重规划但实际执行旧 chunk 均为关键失败。

## 9. 常见错误与排查

- chunk 从 `t+1` 开始：明确第 0 个应为当前 `a_t`。
- episode 末尾用下一 episode 动作补齐：索引需先按 episode 截断。
- 每步预测但仍执行旧 chunk：日志记录 prediction_id 和 offset。
- 比较策略使用不同扰动：固定同一事件和初态。

## 10. 时间不足时的最低完成线

完成 T=5/H=3 全部标签、valid 和 masked loss；口头推演两种执行策略。

## 11. 可选提高任务

实现指数权重 temporal ensemble，列出同一环境时刻来自哪些历史 chunk；先用合成动作验证索引，不急于训练。

## 12. 完成后的自测题

1. action chunk 第一个标签对应哪个时刻？
2. chunk 越长为何更可能积累开环误差？
3. 训练 horizon 与执行重规划间隔是否必须相等？
4. 尾部 valid mask 忽略的是什么？

[← 上日：Sequence Dataset](day-03.md) · [周首页](README.md) · [次日：Transformer Policy →](day-05.md)
