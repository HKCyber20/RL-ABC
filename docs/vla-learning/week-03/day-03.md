# 第 3 周 Day 3：损失、动作归一化与混合动作头

[填写本日 Notebook 作业](../notebooks/week-03/day-03.ipynb)

[← 上日：BC Dataset](day-02.md) · [周首页](README.md) · [次日：数据划分 →](day-04.md)

## 1. 今日目标与预计时间

预计 100–130 分钟。理解 MSE/MAE 与离散夹爪损失，正确拟合训练集归一化统计，按动作维度报告误差并完成反归一化验证。

## 2. 今日知识点及其在 VLA 中的作用

平移（米）、旋转（弧度）、夹爪（离散/连续）的尺度和语义不同，直接平均 MSE 可能让大尺度维度主导训练。归一化改善优化，但统计只能来自训练集，执行前必须反归一化。夹爪若为二元事件，分类 loss 往往比连续 MSE 更符合语义。

## 3. 必要概念、公式、形状与数据流

连续动作回归：

\[
L_{cont}=\frac1{6}\sum_{j=1}^6 w_j(\hat a_j-a_j)^2
\]

二元夹爪：

\[
L_g=\operatorname{BCEWithLogits}(z_g,g),\quad L=L_{cont}+\lambda_gL_g
\]

z-score：`a_norm=(a-mu)/(sigma+eps)`，`mu,sigma [d_a]` 只由 train transitions 计算；执行使用 `a=a_norm*sigma+mu`。也可用第 2 周的固定界限 min-max，但不可混用统计。

## 4. 分步骤学习

1. 计算一个未归一化 batch 的分维 MSE。
2. 仅用训练数据拟合 `mu,sigma`，处理近零方差维。
3. 验证 normalize/denormalize round trip。
4. 建立 6 维连续 + 1 维夹爪双头输出。
5. 比较总 loss 相似但分维错误不同的两个预测。

## 5. 必做作业

对一个至少 20 条、7 维的教学动作数组计算训练统计；提交每维 mean/std。实现归一化往返和 `mixed_action_loss`：连续前 6 维 MSE、夹爪 BCE。构造两个预测 A/B，使总均方误差接近但一个平移差、一个夹爪错，比较任务意义。验证验证/测试动作不参与统计。

## 6. 输入、预期输出与验证方法

- 输入：`action [N,7]`，夹爪标签为 0/1；train indices 明确。
- 预期：统计 `[7]`；normalized/denormalized `[N,7]`；标量 total loss 以及连续/夹爪分项。
- 验证：往返误差 `<1e-6`；train normalized mean 约 0（近零方差维例外并记录）；BCE 输入是 logits；测试统计未泄漏。

低资源替代：用 6 条二维动作 + 二元夹爪，在表格中手算 mean/std 和分项损失。

## 7. 提交内容与固定格式

固定包含：`动作语义/单位`、`train统计`、`归一化往返`、`loss公式与代码`、`A/B分维比较`、`泄漏检查`、`投入分钟数`。

## 8. 评分与验收标准

100 分，80 分通过：统计来源 20；往返 20；混合 loss 25；分维指标 20；任务解释 15。用全数据统计、把 sigmoid 后概率送入 logits loss、忽略单位/夹爪语义均为关键失败。

## 9. 常见错误与排查

- std 为 0 导致 NaN：设 epsilon 或固定该维并记录。
- 验证集重新拟合统计：必须复用训练统计。
- 同时对夹爪 z-score 再 BCE：先确定建模方式。
- 只记录 total loss：至少拆出 translation/rotation/gripper。

## 10. 时间不足时的最低完成线

完成 train-only 统计、连续动作往返和分维 MSE；夹爪双头可登记补救。

## 11. 可选提高任务

比较 MSE、MAE、Huber 在含一个异常动作 batch 上的数值和梯度趋势，写出选择依据。

## 12. 完成后的自测题

1. 为什么不能用测试集估计归一化统计？
2. 不同量纲的动作直接平均 MSE 有何风险？
3. BCEWithLogits 输入为什么不先 sigmoid？
4. 反归一化应该发生在模型内部还是执行适配层？两者如何保证一致？

[← 上日：BC Dataset](day-02.md) · [周首页](README.md) · [次日：数据划分 →](day-04.md)
