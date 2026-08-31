# 第 4 周 Day 5：最小 Transformer Policy

[填写本日 Notebook 作业](../notebooks/week-04/day-05.ipynb)

[← 上日：Action Chunk](day-04.md) · [周首页](README.md) · [次日：频率与延迟 →](day-06.md)

## 1. 今日目标与预计时间

预计 2–3 小时。实现一个读取历史状态/目标 token 并输出 action chunk 的小型 Transformer policy，完成 padding mask、前向、masked loss 和 backward 验证。

## 2. 今日知识点及其在 VLA 中的作用

Transformer policy 将时序观测编码为上下文，再用池化或 action query 预测多个未来动作。真实 VLA 可把状态 token 换成视觉/语言 token；今天重点是接口、mask 与时序输出，而非扩大模型。

## 3. 必要概念、公式、形状与数据流

采用 `B=4,H_o=6,d_s=2,d_goal=2,D=32,H_a=4,d_a=2`：

```text
state_history [4,6,2] -> state_proj -> [4,6,32]
goal          [4,2]   -> goal_proj  -> [4,1,32]
concat tokens                       -> [4,7,32]
+ position -> TransformerEncoder(mask) -> [4,7,32]
pooled valid history / goal token -> [4,32]
action_head -> [4,H_a*d_a] -> [4,4,2]
target/mask -> masked MSE -> scalar -> backward
```

若使用 action query，输出可直接 `[B,H_a,D]` 后投影；本课必做允许简单 flatten head。编码历史不一定需要 causal mask，因为所有历史均已发生，但必须使用 padding mask。

## 4. 分步骤学习

1. 实现 state/goal projection 与位置 embedding。
2. 拼接时同步扩展 valid mask；goal token 始终有效。
3. 运行 1–2 层小 TransformerEncoder。
4. 只对有效历史池化或读取 goal token。
5. 输出 chunk，计算 action valid masked MSE。
6. backward 并检查 state projection、encoder、action head 梯度。

## 5. 必做作业

实现 `TinyChunkPolicy`，输入上述固定形状并输出 `[4,4,2]`。构造不同 history lengths `[6,4,2,5]` 和对应 mask；至少 6 个 shape/mask 断言；执行 forward/backward/optimizer step。反事实验证：只修改 padding 状态为极大数，eval 模式下输出应近似不变；修改有效状态应改变输出。

## 6. 输入、预期输出与验证方法

- 输入：state `[4,6,2]`、goal `[4,2]`、history_valid `[4,6]`、target `[4,4,2]`、action_valid `[4,4]`。
- 预期：prediction `[4,4,2]`、有限标量 loss、三模块非空有限梯度。
- 验证：padding 反事实输出差异小于合理浮点容差；有效输入反事实差异非零；step 后参数变化；固定 seed 可复现。

低资源方案：`B=2,D=8,nhead=2,1 layer,H_o=3,H_a=2` CPU；若无法训练，只做一次 forward/backward 仍可完成当日最低线。

## 7. 提交内容与固定格式

固定包含：`模型数据流/形状`、`mask语义`、`代码`、`断言`、`loss/梯度/step日志`、`padding反事实`、`参数量与资源`、`投入分钟数`。

## 8. 评分与验收标准

100 分，80 分通过：结构形状 20；padding mask 20；action mask loss 15；forward/backward 20；反事实 15；复现/资源 10。输出形状错误、padding 影响输出明显却未解释、梯度不到 encoder 任一为关键失败。

## 9. 常见错误与排查

- goal token mask 成 padding：拼接后最后位置必须有效。
- position embedding 长度不含 goal：序列实际为 `H_o+1`。
- `src_key_padding_mask` True 语义与 valid 相反：用 Day 3 可控测试确认。
- train 模式 dropout 让反事实不稳定：对比时 `model.eval()` 且 no_grad。
- action mask `[B,H_a]` 未广播动作维：显式扩展或先按 d_a 平均。

## 10. 时间不足时的最低完成线

用最小配置完成输出 `[B,H_a,d_a]`、正确 masked loss、backward 和 padding 反事实。

## 11. 可选提高任务

改为 learnable action queries 通过 cross-attention 读取编码历史，比较权重形状 `[B,heads,H_a,H_o+1]` 与 flatten head 的差异。

## 12. 完成后的自测题

1. 为什么只输入历史时 encoder 不一定需要 causal mask？
2. history mask 与 action mask 分别作用在哪里？
3. padding 反事实为什么比只看 shape 更强？
4. 输出 H_a 个动作是否意味着必须一次全执行？

[← 上日：Action Chunk](day-04.md) · [周首页](README.md) · [次日：频率与延迟 →](day-06.md)
