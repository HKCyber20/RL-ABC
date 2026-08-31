# 第 8 周 Day 2：连续动作回归与动作块标签

[填写本日 Notebook 作业](../notebooks/week-08/day-02.ipynb)

[上一课：Day 1](day-01.md) · [本周首页](README.md) · [下一课：Day 3](day-03.md)

## 1. 今日目标与预计时间

预计 120 分钟。实现连续 action chunk head、逐类 loss、normalization/denormalization 往返和序列尾部 mask，为闭环主线建立无歧义动作输出。

## 2. 这在 VLA 中的作用

多数操作 VLA 最终要控制连续位置、旋转和夹爪。动作尺度、chunk 标签偏移或尾部 padding 处理错误，会让模型在离线训练看似收敛却执行错误时间步。

## 3. 核心概念、公式与张量

标签块：

\[
A_t=[a_t,a_{t+1},\ldots,a_{t+H-1}]\in\mathbb R^{H\times d_a}.
\]

batch 为 `[B,H,d_a]`，有效 mask `[B,H]`。损失：

\[
L=\frac{\sum_{b,h}m_{bh}\sum_j w_j\ell(\hat a_{bhj},a_{bhj})}{\sum_{b,h}m_{bh}}.
\]

位置/旋转可用 MSE/Huber，二值 gripper 用 BCE logits；切分 action head 时必须记录逐维索引。

## 4. 分步学习

1. 用明确例子写 `obs_t` 对应的 chunk 起点，确认包含 `a_t` 而非 `a_{t+1}`。
2. Dataset 在 episode 尾部生成 padding 与 `chunk_valid_mask`。
3. action head 输出 `[B,H,d_a]` 或连续项 + gripper logits。
4. 仅用 train 统计归一化；逐维 round-trip 测试。
5. 用手算小 batch 验证 masked loss，不让 padding 影响均值。
6. 做 16–32 样本过拟合并检查每个 horizon 的误差。

## 5. 必做作业

- 提交 chunk 标签时间线和尾部 padding 策略。
- 实现连续 head 与 masked loss，至少通过 shape、手算 loss、全/部分 mask、往返、梯度五项测试。
- 输出逐 horizon、逐动作维 MAE，不能只看总 loss。

## 6. 输入、预期输出与验证

输入：`B=3,H=4,d_a` 按 schema；其中一条 episode 尾部仅前 2 步有效。

预期输出：预测 `[3,4,d_a]`；mask `[3,4]`；改变无效 padding 标签不改变 loss；反归一化恢复原动作。均为预期结果。

验证：手工构造小 tensor 算出期望损失；全无效样本按契约拒绝；动作进入环境前检查物理范围但记录未 clip 原值和修正值。

## 7. 固定提交格式

```markdown
# W8D2 提交
- 实际用时：
## Chunk 标签时间线
## 动作逐维/head/loss 定义
## 五项自动测试
## 小批过拟合与逐 horizon 指标
## 反归一化/边界检查日志
## 自测答案
```

## 8. 评分与验收

- 标签索引和尾部 mask 正确：25 分。
- 连续 head 与动作分项合理：20 分。
- masked loss 手算/负向测试：20 分。
- normalization 往返与边界记录：20 分。
- 小批指标和自测：15 分。

总分 100，80 分通过；标签偏移、padding 进入 loss、动作统计含 val/test 或夹爪 logits 被当物理值执行时不通过。

## 9. 常见错误与排查

| 错误 | 排查 |
|---|---|
| chunk 学成滞后一帧 | 可视化 `obs_t,a_t,a_{t+1}` 并做已知序列单测 |
| mask 后 loss 仍变化 | 先逐元素 loss，再乘 mask，分母只计有效元素 |
| gripper 始终多数类 | 分开 BCE/accuracy，检查类别比例和权重 |
| 旋转误差主导 | 用合理表示/尺度并分项记录，不盲目统一权重 |

## 10. 时间不足时的最低完成线

用 `B=2,H=3,d_a=3` 的 CPU tensor 通过标签、mask、手算 loss、round-trip 和 backward；不要求长训练。

## 11. 可选提高

比较 MSE/Huber；用 6D rotation 或 quaternion 专用损失；为 horizon 设置衰减权重并做消融。

## 12. 完成后的自测题

1. `obs_t` 的第一个动作标签应是哪一步？
2. 为什么 episode 尾部需要 mask？
3. gripper logits 如何变成环境命令？
4. 逐 horizon 指标能发现什么？

[上一课：Day 1](day-01.md) · [本周首页](README.md) · [下一课：Day 3](day-03.md)
