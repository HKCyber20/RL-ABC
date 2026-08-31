# 第 4 周 Day 1：图像、状态、动作的时间对齐

[填写本日 Notebook 作业](../notebooks/week-04/day-01.ipynb)

[← 上日：低维 BC 周项目](../week-03/day-07.md) · [周首页](README.md) · [次日：历史帧 →](day-02.md)

## 1. 今日目标与预计时间

预计 90–120 分钟。建立传感器时间戳与控制时间的对齐规则，实现 nearest/previous 对齐并检测标签整体偏移一帧。

## 2. 今日知识点及其在 VLA 中的作用

相机、机器人状态和控制器可能以不同频率、不同延迟记录。VLA 样本必须表示“策略在决策时实际可见的信息”以及“随后执行的动作”。把未来帧配给当前动作会泄漏，把过去动作配给当前观测会产生滞后标签。

## 3. 必要概念、公式、形状与数据流

决策时间 `t_k` 的样本可定义为：

```text
image = latest image timestamp <= t_k
state = interpolated/latest state at t_k
label = command issued for interval [t_k, t_{k+1})
```

对来源时间 `u_i` 与目标 `t_k`，nearest 为 `argmin_i |u_i-t_k|`，但若不允许未来信息，应使用 previous：`max{i:u_i<=t_k}`。记录 age：`age=t_k-u_i`，若超过阈值丢弃/标无效。

形状：原始 image timestamps `[N_i]`、state `[N_s,d_s]`、action `[T,d_a]`；对齐后 `image_idx/state_idx [T]` 和 valid `[T]`。

## 4. 分步骤学习

1. 画相机 10 Hz、状态 50 Hz、动作 20 Hz 的 0.5 秒时间轴。
2. 明确允许/禁止未来采样，选择 previous 或 nearest。
3. 实现索引与 age 计算。
4. 构造 known dynamics 检查 `s_{t+1}=s_t+a_t`。
5. 将 action 滚动一帧，比较动力学残差与闭环行为。

## 5. 必做作业

用给定频率生成不含真实图像内容的时间戳序列，动作时间 0–0.5 s。实现 previous 对齐并输出每个动作对应的 image/state index 与 age；设最大 image age 0.12 s。再对第 3 周一条轨迹生成正确标签和 `shift(+1)` 标签，计算两者一步动力学残差。明确边界动作如何处理，禁止循环 roll 把末项接到首项。

## 6. 输入、预期输出与验证方法

- 输入：camera 10 Hz、state 50 Hz、action 20 Hz 的时间戳；一条已知动力学轨迹。
- 预期：索引单调不减；previous 时间不晚于决策时刻；age 非负且 valid 样本不超过阈值；错位标签残差通常更大，具体数值以数据为准。
- 验证：断言 `source_ts[idx] <= target_ts + eps`；边界无负索引误取末帧；报告 valid 比例。

低资源方案：在纸上对齐前 6 个动作时刻并手算残差。

## 7. 提交内容与固定格式

固定包含：`采样约定`、`时间轴`、`对齐算法`、`索引/age表`、`valid断言`、`一帧错位残差`、`边界策略`、`投入分钟数`。

## 8. 评分与验收标准

100 分，80 分通过：约定 20；对齐实现 25；age/valid 15；错位实验 25；边界 15。读取未来帧却称在线可用、循环 roll、动作与区间语义不明均为关键失败。

## 9. 常见错误与排查

- 浮点时间直接等号匹配：用 searchsorted/容差。
- nearest 偷看未来：若是离线重建也要标出与部署不一致。
- 相机 delay 未计入 timestamp：区分采集时间、到达时间。
- shift 后长度不一致时随意填 0：边界应剔除或 valid mask。

## 10. 时间不足时的最低完成线

完成 6 个动作时刻的 previous 对齐、age 检查和一个不循环的错位反例。

## 11. 可选提高任务

比较 previous、nearest 和线性插值对 state 的误差；解释为何图像通常不能直接数值线性插值。

## 12. 完成后的自测题

1. 何时 nearest 会引入未来信息？
2. image age 为什么应进入数据质量报告？
3. `a_t` 控制的是哪个时间区间？
4. 动力学残差能否发现所有错位？它有什么局限？

[← 上日：低维 BC 周项目](../week-03/day-07.md) · [周首页](README.md) · [次日：历史帧 →](day-02.md)
