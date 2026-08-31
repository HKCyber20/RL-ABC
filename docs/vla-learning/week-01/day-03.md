# 第 1 周 Day 3：Q/K/V 与 Scaled Dot-Product Attention

[填写本日 Notebook 作业](../notebooks/week-01/day-03.ipynb)

[← 上日：Token 与位置](day-02.md) · [周首页](README.md) · [次日：Mask 与多头 →](day-04.md)

## 1. 今日目标与预计时间

预计 90–120 分钟。能从形状和数值两方面解释 Q、K、V，手写 scaled dot-product attention，并验证每个 query 对所有 key 的权重和为 1。

## 2. 今日知识点及其在 VLA 中的作用

Q 表示“当前 token 要查什么”，K 表示“每个候选如何被匹配”，V 是匹配后取回的信息。VLA 用这套机制让语言或动作查询选择性读取视觉区域。缩放防止维度较大时点积使 softmax 过饱和。

## 3. 必要概念、公式、形状与数据流

\[
Q=X_QW_Q,\quad K=X_KW_K,\quad V=X_VW_V
\]

\[
S=\frac{QK^T}{\sqrt{d_k}},\quad A=\operatorname{softmax}(S,\text{dim}=-1),\quad O=AV
\]

通用形状：

```text
Q [B,N_q,d_k]
K [B,N_k,d_k]
V [B,N_k,d_v]
S,A [B,N_q,N_k]
O [B,N_q,d_v]
```

softmax 在 `N_k` 维进行，因为每个 query 要在所有 key 之间分配读取比例。

## 4. 分步骤学习

1. 先仅检查矩阵乘法形状，写出 `Q @ K^T` 的输出。
2. 用 `B=1,N_q=2,N_k=3,d_k=2,d_v=2` 的整数小矩阵手算 scores。
3. 用代码计算 softmax，打印行和。
4. 计算 `A@V`，解释输出是 value 的加权和。
5. 放大一个 key 与某 query 的相似度，比较权重变化。

## 5. 必做作业

实现函数 `attention(q,k,v,mask=None)`，不调用框架的整层 attention。使用固定输入：

```python
q = [[[1., 0.], [0., 1.]]]
k = [[[1., 0.], [0., 1.], [1., 1.]]]
v = [[[1., 0.], [0., 2.], [3., 3.]]]
```

提交 scores、weights、output 及形状；断言 `weights.sum(-1)` 接近全 1。再把第三个 key 改为 `[3,0]`，解释第一个 query 权重如何变化。具体小数以实际运行结果为准，不预先伪造。

## 6. 输入、预期输出与验证方法

- 输入形状：Q `[1,2,2]`、K/V `[1,3,2]`。
- 预期形状：scores/weights `[1,2,3]`、output `[1,2,2]`。
- 预期性质：权重非负、最后一维和约为 1、输出有限；修改 K 后至少一个权重发生变化。
- 验证：`assert torch.allclose(weights.sum(-1), ones, atol=1e-6)`；检查 `isfinite`；手算第一行点积用于交叉核对。

低资源替代：用计算器完成上述固定矩阵，softmax 保留三位小数。

## 7. 提交内容与固定格式

提交 Markdown，固定包含：`QKV 直觉`、`形状推导`、`实现`、`原始运行结果`、`修改 key 后对比`、`断言证据`、`投入分钟数`。

## 8. 评分与验收标准

100 分，80 分通过：公式与缩放 15；形状 20；实现 25；权重行和验证 20；干预实验解释 20。softmax 维度错误、忘记转置 K 或输出不是 `A@V` 属关键项失败。

## 9. 常见错误与排查

- `K.transpose(0,1)` 错换 batch：只交换最后两维。
- 对 query 维 softmax：打印 `weights.sum(-1)` 定位。
- 使用整数张量进入 softmax：转浮点。
- 用 `sqrt(N_k)` 缩放：分母应为 key 的特征维 `sqrt(d_k)`。
- 修改 K 后输出没变：确认变量真正进入重新计算而非复用旧 weights。

## 10. 时间不足时的最低完成线

手算第一行 scores，写出所有形状，并验证一行 softmax 权重和为 1。

## 11. 可选提高任务

比较不缩放与缩放情况下，当 Q/K 同时乘 10 后的权重熵，说明过度尖锐的 softmax 对梯度的影响。

## 12. 完成后的自测题

1. 为什么 K 和 V 的序列长度必须相同？
2. Q 与 K 的最后一维为什么必须兼容？
3. 输出序列长度跟随 Q 还是 V？
4. attention 权重是否等于完整的因果解释？为什么不应这样断言？

[← 上日：Token 与位置](day-02.md) · [周首页](README.md) · [次日：Mask 与多头 →](day-04.md)
