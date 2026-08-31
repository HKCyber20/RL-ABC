# 第 6 周 Day 4：视觉 BC 训练流水线

[填写本日 Notebook 作业](../notebooks/week-06/day-04.ipynb)

[上一课：Day 3](day-03.md) · [本周首页](README.md) · [下一课：Day 5](day-05.md)

## 1. 今日目标与预计时间

预计 2–3 小时。建立无数据泄漏、能小批过拟合、能保存/重载且可复现的视觉 BC 训练流水线。

## 2. 这在 VLA 中的作用

训练脚本是数据、模型和动作定义的交汇点。先通过小批过拟合与 checkpoint 一致性，可以在长训练前发现标签错位、归一化错误和参数未更新。

## 3. 核心概念、公式与数据流

按 episode 切分：同一 episode 的相邻帧不能跨 train/val。连续动作基本损失：

\[
\mathcal L=\frac1B\sum_i\sum_j w_j(\hat a_{ij}-a_{ij})^2.
\]

夹爪为二值时可拆为位置/旋转 MSE 与 gripper BCE。训练数据流：`manifest → episode split → train statistics → Dataset → batch → policy → loss → optimizer → checkpoint`。

## 4. 分步学习

1. 固定 seed，按 episode_id 划分 train/val/test 并保存 split 文件。
2. 仅用 train 计算 state/action normalization。
3. 先取 32 个样本反复训练，确认 loss 明显下降并能近似记忆。
4. 再执行短训练，记录 train/val 总 loss 与逐动作维 MAE。
5. checkpoint 保存 model、optimizer、epoch、config、statistics、schema_version 与 git/代码标识。
6. 重载后对固定 batch 比较输出。

## 5. 必做作业

- 实现 episode-level split 检查，断言三个集合 episode_id 无交集。
- 完成 32 样本过拟合测试；若失败，提交定位证据和最小修复。
- 保存 checkpoint 并在新进程或重新构造模型后重载，固定输入输出一致。

## 6. 输入、预期输出与验证

输入：第 5 周数据；CPU 路径使用最多 200 个 `64×64` synthetic 样本、tiny CNN、10–100 个短 epoch。

预期输出：小批 loss 显著低于初值；短训练产生 train/val 曲线；重载输出与保存前 `allclose`。不规定虚构阈值，需报告实际曲线与硬件耗时。

验证：标签随机打乱时不应同样快速泛化；split 交集为空；每次反向更新前清梯度；checkpoint 缺 statistics 时加载器必须拒绝。

## 7. 固定提交格式

```markdown
# W6D4 提交
- 实际用时与设备：
## Episode split 与泄漏检查
## Loss/动作头定义
## 32 样本过拟合曲线
## 短训练指标
## Checkpoint 重载命令与输出差
## 失败排查及自测答案
```

## 8. 评分与验收

- episode split 与 train-only statistics：20 分。
- 小批过拟合有实际证据：25 分。
- 短训练指标完整：15 分。
- checkpoint 内容和重载一致：25 分。
- 负向测试、自测和复现信息：15 分。

总分 100，80 分通过；数据泄漏、没有实际训练日志或 checkpoint 缺动作统计时不通过。

## 9. 常见错误与排查

| 错误 | 排查 |
|---|---|
| loss 完全不降 | 检查 label 时序、optimizer 参数、梯度、lr 和 normalize |
| train 降 val 不降 | 检查 episode split、数据量和过拟合 |
| 重载输出不同 | 设置 eval、加载 statistics、固定预处理和模型配置 |
| gripper 主导 loss | 分项记录并合理设权重/独立分类头 |

## 10. 时间不足时的最低完成线

CPU 上完成 episode split、32 样本过拟合和 checkpoint 重载；完整训练可留给 Day 7，但这三项不能省略。

## 11. 可选提高

加入学习率 scheduler、gradient clipping、早停；比较 MSE 与 Huber；记录实验配置哈希。

## 12. 完成后的自测题

1. 为什么随机帧切分会高估泛化？
2. 小批过拟合失败优先查哪三项？
3. checkpoint 为什么必须带 normalization statistics？
4. 二值夹爪为何可能不适合 MSE？

[上一课：Day 3](day-03.md) · [本周首页](README.md) · [下一课：Day 5](day-05.md)
