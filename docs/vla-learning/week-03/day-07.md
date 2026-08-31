# 第 3 周 Day 7：周项目——低维到达任务行为克隆

[填写本日 Notebook 作业](../notebooks/week-03/day-07.ipynb)

[← 上日：Rollout 评估](day-06.md) · [周首页](README.md) · [次日：第 4 周时间对齐 →](../week-04/day-01.md)

## 1. 今日目标与预计时间

预计 3–4 小时。完成从专家示范、Dataset、无泄漏划分、归一化、训练到 20+ 次闭环评估的最小 BC 项目，并以失败证据而非只看 loss 复盘。

## 2. 今日知识点及其在 VLA 中的作用

这是第一个完整策略学习闭环。虽然输入仅是低维状态/目标而非图像语言，但数据工程、动作接口、训练划分、checkpoint 和 rollout 评估与真实 VLA 同构。先把这条链路做可靠，后续只替换/扩展观测编码。

## 3. 必要概念、公式、形状与数据流

建议二维任务：

```text
input = concat(position [B,2], goal [B,2]) -> [B,4]
MLP -> normalized action [B,2]
denormalize/clip -> environment
state[t+1] = state[t] + action[t] (+ optional noise)
```

BC loss `MSE(pred_norm,target_norm)`；评估成功条件固定为终点距离阈值与 horizon。训练/验证/test episode id 不相交。

## 4. 分步骤学习

1. 闭卷回答周测，发现概念缺口。
2. 由解析专家生成多个 episode，保存 schema 和 seed。
3. 按 episode 划分，拟合 train-only 统计。
4. 训练 MLP/线性基线，保存 checkpoint 与曲线。
5. 在固定 test 初态执行至少 20 次 rollout，并运行 zero/random 或专家基线。
6. 分类失败，写出下一周加入时序的动机。

## 5. 必做作业

周测（25 分）：解释 transition 对齐、episode 划分、train-only normalization、协变量偏移、rollout 成功率。

周项目（75 分）：提交可复现的二维到达 BC。至少 30 条专家 episode；明确 train/val/test id；模型可为小 MLP；记录 train/val loss；test 至少 20 rollout；给出基线、成功率、最终距离和失败类型。若无 PyTorch，用 NumPy 线性回归完成，仍需同样数据与闭环评估。

## 6. 输入、预期输出与验证方法

- 输入：固定随机种子、专家控制器、30+ episode、动作规范。
- 预期：数据/模型/评估产物齐全；训练可结束并输出有限 loss；具体成功率是实验结果，不设预期。
- 验证：从干净环境按提交命令复跑；split 无交集；反归一化 round trip；测试原始记录可重算汇总；checkpoint 与配置对应。

## 7. 提交内容与固定格式

固定包含：`周测`、`问题与动力学`、`数据schema和split`、`模型/训练配置`、`loss曲线或表`、`20+ rollout原始记录与汇总`、`基线`、`失败分析`、`复跑命令`、`总投入时间`。

## 8. 评分与验收标准

100 分，80 分通过：周测 25；数据/对齐 15；无泄漏/统计 15；训练证据 15；rollout/基线 20；失败与复现 10。四个门槛：时间对齐、episode 无泄漏、动作反归一化、闭环成功率。策略性能差但诊断完整可通过；伪造或挑选结果不通过。

## 9. 常见错误与排查

- expert 与 environment 的 dt 不一致：用一条示范逐步回放。
- 训练 loss 正常而动作很小：检查归一化/反归一化。
- test 初态出现在 train：审计 episode/seed/目标位置。
- rollout 调用 target action 而非 prediction：在执行点打印来源并做 zero-policy 对照。
- 只保存最终 checkpoint：同时保存配置、统计和 split。

## 10. 时间不足时的最低完成线

使用线性策略完成 30 条数据、无泄漏划分、训练和 20 次 rollout；神经网络与图表可后补，但证据缺失时整周暂不勾选。

## 11. 可选提高任务

比较 MLP 与线性模型，或加入训练范围外目标位置，报告 IID/OOD 两组成功率与失败差异。

## 12. 完成后的自测题

1. 能否从任一 test 失败追溯到 seed、checkpoint、动作和状态序列？
2. 哪个证据证明 validation 没有 episode 泄漏？
3. 归一化文件丢失后 checkpoint 是否还能正确执行？
4. 下一周为什么要用历史帧或 action chunk？

[← 上日：Rollout 评估](day-06.md) · [周首页](README.md) · [次日：第 4 周时间对齐 →](../week-04/day-01.md)
