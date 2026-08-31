# 第 7 周 Day 4：Cross-Attention 视觉语言融合

[填写本日 Notebook 作业](../notebooks/week-07/day-04.ipynb)

[上一课：Day 3](day-03.md) · [本周首页](README.md) · [下一课：Day 5](day-05.md)

## 1. 今日目标与预计时间

预计 2–3 小时。实现一个文本查询图像 patch 的 cross-attention 模块，验证 Q/K/V 来源、mask、张量形状、权重归一化和梯度。

## 2. 这在 VLA 中的作用

Cross-attention 允许指令按需读取不同图像区域，例如“红色方块”和“蓝色盒子”对应不同 patch。它提供可学习的信息路由，但权重本身不是因果解释。

## 3. 核心概念、公式与张量

\[
Q=X_{text}W_Q,\quad K=X_{img}W_K,\quad V=X_{img}W_V,
\]

\[
A=softmax(QK^T/\sqrt{d_h}),\quad O=AV.
\]

若 `B=4,h=4,L=12,N=64,d_h=32`：`Q [4,4,12,32]`、`K/V [4,4,64,32]`、`A [4,4,12,64]`、输出文本 token `[4,12,128]`。文本 pad mask 应让 pad query 不参与最终 pooling；图像有效 mask 用于可变 patch 时屏蔽 K/V。

## 4. 分步学习

1. 让视觉 encoder 保留 spatial tokens `[B,N,D]`，不要过早 global pool。
2. 文本 encoder 输出 token features `[B,L,D]` 与 valid mask。
3. 用手写单头或 `MultiheadAttention(batch_first=True)` 实现 text Q、image K/V。
4. 断言 attention 最后维对有效 patch 求和约为 1；检查 pad token 的 pooled 贡献为 0。
5. 将 cross-attended text 做 masked pooling，与 state 融合输出动作。
6. 用两个 synthetic 彩色 patch 和两条查询做训练/构造实验，观察权重与动作变化，但不把权重单独当 grounding 结论。

## 5. 必做作业

- 画 Q/K/V 来源和所有 shape；实现 cross-attention forward/backward。
- 写 6 个测试：输出 shape、权重 shape、权重和、mask、batch 重排、三分支梯度。
- 与 Day 3 concat 在相同小 batch 上比较参数量、单步耗时和能否过拟合。

## 6. 输入、预期输出与验证

输入：`image_tokens [2,16,64]`、`text_tokens [2,6,64]`、一条文本含 2 个 pad；state `[2,d_s]`。

预期输出：attention weights `[2,h,6,16]` 或库返回的等价聚合形状；有效 query 的 patch 权重和约 1；动作 `[2,d_a]`；梯度可到 Q/K/V 投影。均为预期结果。

验证：故意将 `K` batch 打乱应改变输出；全 mask 样本按契约拒绝而不是产生 NaN；检查库 API 的 mask True 语义和返回权重是否已跨 head 平均。

## 7. 固定提交格式

```markdown
# W7D4 提交
- 实际用时与设备：
## Q/K/V 来源和 shape 图
## Cross-attention 实现说明
## 六个自动测试及日志
## 彩色 patch 查询实验
## 与 concat 的小样本对照
## 自测答案
```

## 8. 评分与验收

- Q/K/V 来源与 shape 全对：20 分。
- mask 与权重归一化正确：25 分。
- forward/backward 和六项测试：25 分。
- concat 公平小对照：15 分。
- 解释边界与自测：15 分。

总分 100，80 分通过；softmax 维错误、mask 语义不明、全 mask NaN 或把 attention 图当作唯一 grounding 证据时不通过。

## 9. 常见错误与排查

| 错误 | 排查 |
|---|---|
| 权重 shape 少了 head | 查看 API 是否默认 average_attn_weights |
| pad query 仍影响 pooling | pooling 再应用 valid mask，而非只 mask K/V |
| Q/K/V 维不匹配 | 明确统一 d_model 或分别投影 |
| 内存突然增大 | attention 复杂度与 `L×N` 成正比，先减分辨率/patch 数 |

## 10. 时间不足时的最低完成线

用 `B=2,L=4,N=8,D=32` 的 CPU synthetic token，完成单头 attention、mask、权重和、backward 和动作 shape 测试。

## 11. 可选提高

反向做 image Q、text K/V 并解释差异；画 patch 网格热力图；增加 learnable action query。

## 12. 完成后的自测题

1. text Q、image K/V 表达怎样的信息方向？
2. softmax 应在哪个维度，为什么？
3. mask K/V 与屏蔽 pad query 有何不同？
4. attention 权重为何不是 grounding 的充分证据？

[上一课：Day 3](day-03.md) · [本周首页](README.md) · [下一课：Day 5](day-05.md)
