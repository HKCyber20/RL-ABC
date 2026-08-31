# 第 8 周 Day 5：闭环推理、Action Chunk 与 Receding Horizon

[填写本日 Notebook 作业](../notebooks/week-08/day-05.ipynb)

[上一课：Day 4](day-04.md) · [本周首页](README.md) · [下一课：Day 6](day-06.md)

## 1. 今日目标与预计时间

预计 2–3 小时。实现安全、可记录的闭环推理循环，明确预测 H 步、实际执行 K 步的关系，并比较单步与 receding-horizon action chunk。

## 2. 这在 VLA 中的作用

Action chunk 可减少推理频率并产生更平滑动作，但执行过长会降低反馈纠错能力。Receding horizon 每执行少量动作就重新观察，让策略兼顾时序一致性和闭环响应。

## 3. 核心概念、公式与数据流

策略每次预测：`Â_t=[â_t^0,…,â_t^{H-1}]`。执行前 `K≤H` 个动作，然后重新观测；`K=1` 是完全闭环，`K=H` 是整块开环。

```text
observe → preprocess → predict H actions
→ denormalize → finite/range/slew-rate checks
→ execute first K → log every transition
→ re-observe until success/termination/truncation
```

可选 temporal ensemble：对同一时刻来自不同历史 chunk 的预测按时间衰减加权，但本周不是必做。

## 4. 分步学习

1. 将推理预处理直接复用训练模块，加载并验证 metadata。
2. 每次预测记录 inference_time、原始规范化动作、物理动作、clip/reject 原因。
3. 加有限值、物理边界、单步变化率和夹爪状态检查；Mock 同样执行。
4. 实现 `H=1,K=1` baseline，再实现 `H=4,K=1` 和 `H=4,K=4`。
5. 用相同 seeds 各执行至少 5 次探索性 rollout，比较成功、长度、推理次数和平滑度。
6. 对目标中途轻微扰动，观察 K 的反馈代价。

## 5. 必做作业

- 提交 rollout 伪代码/实现和完整日志 schema。
- 验证 chunk 索引：环境实际执行的每个动作与预测 chunk 的来源一致。
- 比较三种 `(H,K)`；资源不足可用 CPU Mock，但必须 autonomous。
- 注入一个 NaN 或越界预测，安全层应拒绝/安全终止并记录。

## 6. 输入、预期输出与验证

输入：Day 4 checkpoint、固定 5–10 个 seeds、`(1,1),(4,1),(4,4)`；可在 Mock 中移动目标作扰动。

预期输出：每种设置有逐 episode 日志；`K` 越大推理次数通常减少但对扰动响应可能变慢；NaN/越界不进入环境。实际性能不预设。

验证：从日志重建每个执行动作的 chunk id/offset；success checker 独立复算；episode 结束后不再 step；动作修正数可汇总。

## 7. 固定提交格式

```markdown
# W8D5 提交
- 实际用时与设备：
## Rollout 数据流与日志 schema
## 安全检查规则
## (H,K) 对照表
## Chunk 来源/执行索引验证
## 扰动与 NaN/越界故障注入
## 自测答案
```

## 8. 评分与验收

- 闭环循环与训练接口一致：20 分。
- chunk 索引可追溯：20 分。
- 三种 H/K 公平对照：20 分。
- 安全层与故障注入：25 分。
- 扰动分析、自测与日志：15 分。

总分 100，80 分通过；无新观测却称闭环、动作被 silent clip、NaN 进入环境或 chunk offset 无法追溯时不通过。

## 9. 常见错误与排查

| 错误 | 排查 |
|---|---|
| 每步都执行 chunk[0] | 记录 chunk_id/offset 并做已知序列测试 |
| H=4 模型实际只按 H=1 训练 | 检查 checkpoint config 与输出 shape |
| success 后仍执行剩余 chunk | 每个 action 后检查终止/成功 |
| K 对照 seeds 不同 | 固定同一 seed 列表和 reset options |

## 10. 时间不足时的最低完成线

在 CPU Mock 比较 `(1,1)` 与 `(4,1)` 各 5 次，并通过 chunk offset、NaN 拒绝和 success 后停止三项测试。

## 11. 可选提高

实现 temporal ensembling；测控制抖动/jerk；把推理延迟纳入实时周期预算。

## 12. 完成后的自测题

1. H 与 K 分别控制什么？
2. `K=H` 为何更像局部开环？
3. action chunk 怎样降低推理频率？
4. 安全 clip 为什么必须记录原值？

[上一课：Day 4](day-04.md) · [本周首页](README.md) · [下一课：Day 6](day-06.md)
