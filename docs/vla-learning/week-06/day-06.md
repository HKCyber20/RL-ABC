# 第 6 周 Day 6：离线指标与闭环 rollout 的差异

[填写本日 Notebook 作业](../notebooks/week-06/day-06.ipynb)

[上一课：Day 5](day-05.md) · [本周首页](README.md) · [下一课：Day 7](day-07.md)

## 1. 今日目标与预计时间

预计 120 分钟。计算逐动作维离线指标并执行闭环 rollout，用扰动实验展示低验证误差为何不保证高任务成功率。

## 2. 这在 VLA 中的作用

行为克隆只在专家分布上学习。闭环时一次小误差会改变下一帧，模型遇到未见状态后继续偏离；这正是 VLA 项目不能只报告 loss 的原因。

## 3. 核心概念与公式

离线指标：

\[
MAE_j=\frac1N\sum_i|\hat a_{ij}-a_{ij}|,\quad
RMSE_j=\sqrt{\frac1N\sum_i(\hat a_{ij}-a_{ij})^2}.
\]

闭环成功率 `SR = successes / rollouts`，还应报 95% 区间或至少分子/分母。路径指标可含完成阶段、episode 长度、碰撞/越界率。

```text
offline: expert obs → model action vs expert action
rollout: model obs → model action → shifted next obs → ...
```

## 4. 分步学习

1. 在 test episode 上算总 MSE、逐维 MAE、夹爪准确率和动作余弦/方向误差。
2. 固定一组 seed 执行至少 10 次低资源 rollout；主项目目标 20 次。
3. 记录每次 success、最远阶段、长度与首个明显偏离时间。
4. 对初始 state 或目标位置施加小扰动，重复同一策略。
5. 画离线误差与 episode 结果对照，找“低误差但失败”案例。

## 5. 必做作业

- 输出逐维离线指标表，单位使用物理单位或明确规范化尺度。
- 完成固定 seed 的闭环评估；模型尚未训练好时仍需诚实记录失败。
- 选 3 个 rollout 做时间线分析，其中至少一个失败；说明误差如何累积。

## 6. 输入、预期输出与验证

输入：Day 4 checkpoint、held-out episodes、相同预处理/反归一化、10–20 个评估 seed。

预期输出：获得离线指标和成功率两套结果；两者可能相关但不等价。不存在预先保证的成功率，失败结果也是有效证据。

验证：评估脚本 `model.eval()` 且不更新参数；评估 seed 与 train episode 分开；动作进入环境前做有限值与边界检查；人工复核成功判定器。

## 7. 固定提交格式

```markdown
# W6D6 提交
- 实际用时：
## Test 集与评估协议
## 逐动作维离线指标
## Rollout 成功率（分子/分母）
## 扰动对照
## 三个 episode 时间线与失败归因
## 自测答案
```

## 8. 评分与验收

- test 协议无泄漏：15 分。
- 离线指标逐维且单位清楚：20 分。
- 闭环次数、seed 和成功分母清楚：25 分。
- 扰动与时间线分析：25 分。
- 安全检查与自测：15 分。

总分 100，80 分通过；只报 loss、只报百分比不报次数或评估时使用专家纠偏却称 autonomous rollout 时不通过。

## 9. 常见错误与排查

| 错误 | 排查 |
|---|---|
| normalized MAE 被当成米 | 同时报 denormalized 物理单位 |
| eval 仍有随机增强 | 复用确定性 rollout preprocessing |
| 成功率波动大 | 报分子/分母、固定 seeds，增加样本而非挑结果 |
| 首步就异常 | 检查 checkpoint statistics、坐标系与 action clip |

## 10. 时间不足时的最低完成线

CPU Mock 上完成逐维 MAE、10 次 rollout 和 1 个失败时间线；不得只提交离线 loss。

## 11. 可选提高

计算 bootstrap 区间；比较 open-loop replay 与 closed-loop；绘制误差随时间累积曲线。

## 12. 完成后的自测题

1. 为何独立同分布的逐帧误差假设在 rollout 中失效？
2. 夹爪错误为何可能比小位移误差更致命？
3. 成功率为何必须带分子和分母？
4. 专家纠偏 rollout 与 autonomous rollout 有何区别？

[上一课：Day 5](day-05.md) · [本周首页](README.md) · [下一课：Day 7](day-07.md)
