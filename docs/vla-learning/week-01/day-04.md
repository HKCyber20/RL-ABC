# 第 1 周 Day 4：Causal Mask 与 Multi-Head Attention

[填写本日 Notebook 作业](../notebooks/week-01/day-04.ipynb)

[← 上日：Q/K/V](day-03.md) · [周首页](README.md) · [次日：Cross-Attention →](day-05.md)

## 1. 今日目标与预计时间

预计 90–120 分钟。能构造 causal mask，解释为什么自回归模型不能读取未来 token；能推导多头注意力从 `[B,N,D]` 到 `[B,H,N,D_h]` 再合并的形状。

## 2. 今日知识点及其在 VLA 中的作用

Causal mask 约束第 `i` 个位置只能读取不晚于 `i` 的位置，避免训练时偷看未来动作或文本。Multi-head attention 将特征拆成多个子空间，使不同头可学习对象、空间、动作阶段等不同匹配关系；“可学习”不等于每个头必然具有可读语义。

## 3. 必要概念、公式、形状与数据流

长度 `N=4` 的允许矩阵为下三角：

```text
1 0 0 0
1 1 0 0
1 1 1 0
1 1 1 1
```

实现时常将禁止位置在 softmax 前填为 `-inf`。设 `D=32,H=4,D_h=8`：

```text
X              [B,N,32]
Q/K/V 投影      [B,N,32]
split heads    [B,4,N,8]
attention      [B,4,N,N]
head outputs   [B,4,N,8]
concat         [B,N,32]
```

`D` 必须能被头数 `H` 整除。padding mask 与 causal mask 目的不同：前者屏蔽补齐位置，后者屏蔽未来位置。

## 4. 分步骤学习

1. 画 `4×4` mask，逐行解释可见范围。
2. 将 mask 加到 Day 3 的 scores 上，再做 softmax。
3. 用 `B=2,N=5,D=32,H=4` 手推每一步形状。
4. 用 `torch.nn.MultiheadAttention(embed_dim=32,num_heads=4,batch_first=True)` 跑最小样例。
5. 比较无 mask 与 causal mask 下第 1 个位置的输出。

## 5. 必做作业

实现 `make_causal_mask(n)` 并生成 `n=4` mask；把它应用到单头 attention，验证所有未来权重接近 0。再运行多头模块，输入 `[2,5,32]`，打印输出和权重形状。若框架默认返回跨头平均权重，需在记录中说明，并尝试设置不平均的选项或只提交文档推导。

## 6. 输入、预期输出与验证方法

- 输入：随机 `x [2,5,32]`，4 heads；`n=4` 的因果测试张量。
- 预期：MHA 输出 `[2,5,32]`；未平均权重通常为 `[2,4,5,5]`；未来位置权重小于 `1e-6`。
- 验证：检查上三角（不含对角）权重最大值；确认每头 `D_h=8`；输出均为有限数。

低资源替代：完整手画形状图与 `4×4` masked scores，解释每行 softmax 范围。

## 7. 提交内容与固定格式

固定包含：`causal mask 图`、`多头形状链`、`代码或手算`、`未来权重验证`、`框架权重返回说明`、`投入分钟数`。

## 8. 评分与验收标准

100 分，80 分通过：mask 语义 20；mask 实现与验证 25；多头形状 25；框架实验 15；padding/causal 区分 15。若允许未来信息或错误地在 softmax 后加 mask，关键项失败。

## 9. 常见错误与排查

- mask 方向反了：第 0 行只能看第 0 列。
- softmax 后才置零：权重不再归一，应在 softmax 前屏蔽。
- 全行都是 `-inf` 导致 NaN：至少允许当前位置或有效历史。
- 忽略框架的 `batch_first`：检查输入文档与输出第一维。
- 将平均后的 `[B,N,N]` 误报为每头权重。

## 10. 时间不足时的最低完成线

画对 `4×4` causal mask，推导 `B=2,N=5,D=32,H=4` 全部形状，并解释两个 mask 的区别。

## 11. 可选提高任务

同时组合 padding mask 与 causal mask，对长度分别为 3 和 5 的两个序列检查有效权重区域。

## 12. 完成后的自测题

1. 视觉编码器是否总需要 causal mask？
2. 动作序列自回归训练为何需要它？
3. `D=30,H=8` 能直接均分吗？
4. 多头权重被平均后会丢失什么信息？

[← 上日：Q/K/V](day-03.md) · [周首页](README.md) · [次日：Cross-Attention →](day-05.md)
