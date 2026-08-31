# 第 5 周 Day 2：环境 API 与 episode 生命周期

[填写本日 Notebook 作业](../notebooks/week-05/day-02.ipynb)

[上一课：Day 1](day-01.md) · [本周首页](README.md) · [下一课：Day 3](day-03.md)

## 1. 今日目标与预计时间

预计 90 分钟。把昨天的平台或 Mock 包装成稳定的 `reset/step` 契约，并用测试覆盖正常终止、超时截断、非法动作和 seed 复现。

## 2. 这在 VLA 中的作用

VLA 的每个训练样本都来自环境时序。如果把 `obs_t` 与 `obs_{t+1}` 混用、把超时当成功或在 reset 时泄漏上一条轨迹状态，模型会学到错误因果关系。

## 3. 核心概念、公式与数据流

一步转移定义为：

\[
(o_{t+1}, r_t, d_t, u_t, i_t)=\operatorname{step}(a_t\mid o_t),
\]

其中 `d_t=terminated` 表示任务或环境自然终止，`u_t=truncated` 表示时间上限或外部中断。采集记录应明确：`obs_t` 是动作前观测，`action_t` 作用后得到 `next_obs=obs_{t+1}`。

```text
reset → ACTIVE → (success/failure → TERMINATED)
                 (time_limit → TRUNCATED)
任一结束态 → reset → ACTIVE
```

## 4. 分步学习

1. 写出 reset/step 的参数、返回值、dtype 与错误行为。
2. 在 `info` 中统一提供 `is_success`、`step_index`、`termination_reason`。
3. 为每个 episode 生成稳定 `episode_id`，reset 后计数清零。
4. 加动作有限值与边界检查；明确 clip 还是抛错。
5. 写四个最小测试：相同 seed、成功终止、时间截断、结束后禁止继续 step。

## 5. 必做作业

- 创建一页 `env_contract.md`，用表格描述所有接口字段。
- 实现或伪适配 `reset(seed, options)` 和 `step(action)`；真实平台可只写 adapter，不修改第三方源码。
- 运行至少四个契约测试并保留测试名、输入、返回值和断言结果。

## 6. 输入、预期输出与验证

输入：最大步数 `max_steps=5`；一组能到达目标的动作；一组全零动作；一个含 `NaN` 的非法动作。

预期输出：到达目标时 `terminated=True, truncated=False`；全零动作第 5 步 `terminated=False, truncated=True`；非法动作按契约抛出明确异常或被记录为拒绝。均为预期结果。

验证：断言 episode 结束后再次 step 失败；reset 后 `step_index==0`；相同 seed 的初始状态相等；`terminated` 与 `truncated` 不同时为真，除非契约明确说明特殊情况。

## 7. 固定提交格式

```markdown
# W5D2 提交
- 实际用时：
## API 字段表
## 状态机图
## 四个测试的输入与断言
## 运行命令及完整日志
## 失败测试与修复记录
## 自测答案
```

## 8. 评分与验收

- 接口字段、dtype、时间语义：20 分。
- 生命周期与终止原因：20 分。
- 四个契约测试实际通过：30 分。
- 非法动作和 reset 泄漏防护：15 分。
- 记录可复现、自测完整：15 分。

总分 100，80 分通过；混淆 `obs_t/action_t/obs_{t+1}` 或终止类型时不通过。

## 9. 常见错误与排查

| 错误 | 排查 |
|---|---|
| 超时样本被标成功 | success 只由任务判定器产生，不由 `done` 代替 |
| reset 后计数不清零 | 在 reset 测试中检查内部缓存和 episode_id |
| adapter 丢失 info | 明确映射表并保留原始 info 子字段 |
| 最后一步没保存 | 先记录转移，再根据终止标志结束循环 |

## 10. 时间不足时的最低完成线

在 CPU Mock 中通过相同 seed、时间截断、非法动作三个测试，并写清 `obs_t → action_t → obs_{t+1}`。

## 11. 可选提高

增加 context manager；对 100 个随机 seed 做属性测试；测量 `step` 延迟分布并记录 P50/P95。

## 12. 完成后的自测题

1. `terminated` 与 `truncated` 分别能否直接作为成功标签？
2. 为什么最后一步转移也必须落盘？
3. reset 的随机性来源可能有哪些？
4. 何时应该 clip 动作，何时应该拒绝动作？

[上一课：Day 1](day-01.md) · [本周首页](README.md) · [下一课：Day 3](day-03.md)
