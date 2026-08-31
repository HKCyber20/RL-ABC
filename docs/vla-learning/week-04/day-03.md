# 第 4 周 Day 3：Sequence Dataset、Padding 与 Mask

[填写本日 Notebook 作业](../notebooks/week-04/day-03.ipynb)

[← 上日：历史帧](day-02.md) · [周首页](README.md) · [次日：Action Chunk →](day-04.md)

## 1. 今日目标与预计时间

预计 100–140 分钟。将变长历史组成 batch，区分 valid/key-padding/causal mask，正确计算只覆盖有效位置的序列损失。

## 2. 今日知识点及其在 VLA 中的作用

真实 episode 和窗口有效长度不同，batch 需 padding。若模型或 loss 读取 padding，虚构的零动作/零状态会污染训练。VLA 还需区分：padding mask 屏蔽不存在的 token，causal mask 屏蔽未来 token，两者不能互相替代。

## 3. 必要概念、公式、形状与数据流

三个样本有效长度 `[2,4,3]`，pad 到 `S=4`：

```text
x       [B=3,S=4,D]
valid   [3,4]       # True 表示真实位置（本课约定）
padmask [3,4]       # 传部分框架时 True 可能表示屏蔽，需取反并核对 API
causal  [4,4]
```

masked loss：

\[
L=\frac{\sum_{b,t}m_{b,t}\,\ell_{b,t}}{\sum_{b,t}m_{b,t}}
\]

分母是有效元素数；若还含动作维，需明确是先对动作维平均还是 mask 广播后统一平均。

## 4. 分步骤学习

1. 手画长度 2/4/3 的 padded batch 和 valid mask。
2. 实现 collate，返回 padded tensor、valid、lengths。
3. 写 masked mean 并与逐样本循环结果对比。
4. 在 attention 中传 key-padding mask，检查 padding key 权重。
5. 若做自回归，再组合 causal mask 并画允许区域。

## 5. 必做作业

构造三条 `[2,D]`、`[4,D]`、`[3,D]` 序列（`D=2`），实现 padding/collate；创建每位置预测/标签并计算 masked MSE，与逐条不 padding 的 MSE 汇总结果相等。运行或推演一次 attention，验证 padding key 不被读取。提交框架 mask 语义说明，不能只写变量名猜含义。

## 6. 输入、预期输出与验证方法

- 输入：长度 2/4/3、`D=2`。
- 预期：padded `[3,4,2]`、valid `[3,4]`，True 数为 9；masked loss 与参考循环一致。
- 验证：改动 padding 区域的预测值，masked loss 不变；改动有效区，loss 改变；padding key 的注意力权重接近 0。

低资源方案：表格手算 mask 和 loss，完成“改 padding 不变”反事实验证。

## 7. 提交内容与固定格式

固定包含：`mask约定`、`padded batch`、`valid矩阵`、`masked loss公式/代码`、`循环对照`、`注意力屏蔽证据`、`投入分钟数`。

## 8. 评分与验收标准

100 分，80 分通过：padding 20；mask 语义 20；masked loss 25；反事实验证 20；attention 15。True/False 语义反转、分母含 padding、只屏蔽 query 不屏蔽 padding key 均为关键失败。

## 9. 常见错误与排查

- `mean(loss*mask)` 仍除以所有位置：应除以 mask sum。
- 框架 key_padding_mask 中 True 表示忽略，与自定义 valid 相反：先用一条可控样例验证。
- 全 padding 样本导致除零/NaN：数据层拒绝或安全处理并记录。
- causal mask 大小按 batch 而非序列构造。

## 10. 时间不足时的最低完成线

完成 `[3,4,2]` batch、9 个有效位置和 masked loss 与循环一致验证。

## 11. 可选提高任务

实现 length bucketing，估算与随机 batch 相比的 padding 比例；不以牺牲 split 隔离为代价。

## 12. 完成后的自测题

1. padding mask 和 causal mask 各屏蔽什么？
2. 为什么改 padding 数值不应改变 loss？
3. mask 分母为何不能用 `B*S`？
4. attention 中只 mask loss 是否足够？

[← 上日：历史帧](day-02.md) · [周首页](README.md) · [次日：Action Chunk →](day-04.md)
