# 第 3 周 Day 1：专家示范、轨迹与 Episode 契约

[填写本日 Notebook 作业](../notebooks/week-03/day-01.ipynb)

[← 上日：动作接口周项目](../week-02/day-07.md) · [周首页](README.md) · [次日：BC Dataset →](day-02.md)

## 1. 今日目标与预计时间

预计 75–100 分钟。能定义 episode、transition 和 demonstration，画出观测/动作的时间关系，并为低维到达任务设计无歧义数据 schema。

## 2. 今日知识点及其在 VLA 中的作用

模仿学习从专家轨迹学习策略。数据质量首先取决于“这一行代表什么时候”：`a_t` 应与产生它的 `o_t,s_t,l` 对齐，执行后得到 `o_{t+1},s_{t+1}`。若标签整体偏一帧，策略会学到滞后或未来动作，闭环可能振荡。

## 3. 必要概念、公式、形状与数据流

一个长度 T 的 episode：

\[
\tau=\{(o_t,s_t,l,a_t,r_t,done_t)\}_{t=0}^{T-1}\cup(o_T,s_T)
\]

因此通常状态/观测有 `T+1` 个，动作有 `T` 个。语言可作为 episode 级字段，也可复制到每步，但要避免含义不一致。

低维任务 schema 示例：

```text
episode_id: scalar/string
timestamps: [T+1]
state:      [T+1, d_s]
action:     [T, d_a]
instruction: string
success: bool
termination_reason: enum
```

BC 样本取 `(state[t], instruction) -> action[t]`。

## 4. 分步骤学习

1. 区分 episode、trajectory、transition、demonstration。
2. 画 `s_t --a_t--> s_{t+1}` 的 5 步时间轴。
3. 设计字段、dtype、shape、单位、frame 和采样频率。
4. 构造一条长度 4 的二维到达示范。
5. 故意把动作滚动一帧，说明首尾如何处理及为何错误。

## 5. 必做作业

提交一份 `EPISODE_SCHEMA` 表，并手工构造目标 `[1,1]` 的 4 步示范：state `[5,2]`、action `[4,2]`、timestamps `[5]`。验证 `state[t+1]` 与 `state[t]+action[t]` 的关系（本教学环境 `dt=1`、无扰动）。再生成 shifted label，比较对每个 state 的目标动作含义。

## 6. 输入、预期输出与验证方法

- 输入：`s0=[0,0]`、target `[1,1]`、四个自定但能到达目标的二维动作。
- 预期：最终 state 为 `[1,1]`；长度关系 `len(state)=len(action)+1`；时间戳单调。
- 验证：逐步动力学误差接近 0；所有动作单位/frame 在 schema 中声明；shift 后不得将缺失边界静默循环。

低资源方案：纸笔表格即可。

## 7. 提交内容与固定格式

固定包含：`术语定义`、`EPISODE_SCHEMA`、`时间轴`、`4步数据`、`三项断言`、`错位标签分析`、`投入分钟数`。

## 8. 评分与验收标准

100 分，80 分通过：术语 15；schema 25；时间轴 20；示范一致性 20；错位分析 20。状态/动作等长却未说明终态、缺单位/frame、把 `a_t` 对齐到 `s_{t+1}` 作为输入均为关键失败。

## 9. 常见错误与排查

- `done` 与终态混淆：done 属于执行动作后的 transition 标记，需写约定。
- timestamp 用帧号却标秒：记录单位。
- 失败示范和成功示范混用但无字段：保存 success/termination。
- 复制 instruction 时某步不同：优先保存 episode 级真值。

## 10. 时间不足时的最低完成线

完成 schema、5 状态/4 动作时间轴和一次错位解释。

## 11. 可选提高任务

为带 RGB 的未来数据扩展 schema：相机名、分辨率、timestamp、编码格式和外参版本；只设计不采集。

## 12. 完成后的自测题

1. T 个动作为何通常对应 T+1 个状态？
2. `a_t` 的输入应是执行前还是执行后观测？
3. instruction 作为 episode 字段有什么优点？
4. 示范成功字段为什么不能由最后一帧图像猜测代替？

[← 上日：动作接口周项目](../week-02/day-07.md) · [周首页](README.md) · [次日：BC Dataset →](day-02.md)
