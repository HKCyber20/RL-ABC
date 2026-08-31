# 第 5 周 Day 3：观测空间、动作空间与 schema

[填写本日 Notebook 作业](../notebooks/week-05/day-03.ipynb)

[上一课：Day 2](day-02.md) · [本周首页](README.md) · [下一课：Day 4](day-04.md)

## 1. 今日目标与预计时间

预计 90–120 分钟。冻结一版机器可检查的观测/动作 schema，并对形状、dtype、范围、单位、坐标系和时间语义做自动验证。

## 2. 这在 VLA 中的作用

视觉、状态和动作最终会在 batch 维上对齐。schema 是环境、数据加载器、模型和控制器之间的合同；少一个维度说明或单位，就可能让训练 loss 正常而 rollout 完全失败。

## 3. 核心概念、公式与张量

建议单步逻辑字段：

```text
rgb:         uint8  [H, W, 3]       相机坐标定义见元数据
state:       float32 [d_s]           base frame；m/rad
action:      float32 [d_a]           Δpose in base frame；m/rad
instruction: utf-8 scalar
timestamp:   float64 scalar          单调秒
```

训练 batch 常转为 `rgb [B,3,H,W]`、`state [B,d_s]`、`action [B,d_a]`。连续动作归一化可用 `\hat a=(a-\mu)/(\sigma+\epsilon)` 或按物理边界缩放；归一化统计只能来自训练集。

## 4. 分步学习

1. 枚举 observation dict 的所有 key，区分模型输入与仅用于诊断的字段。
2. 对每一维 state/action 命名，例如 `ee_x_m`、`gripper_cmd`。
3. 写清 base/world/camera/tool frame，旋转表示与四元数顺序。
4. 定义动作范围和越界策略；加入 `np.isfinite`。
5. 编写 `validate_step(record)` 并对正常、错 shape、错 dtype、越界、时间倒退五种记录测试。

## 5. 必做作业

- 完成 `trajectory_schema.md` 字段表和一条具名维度示例。
- 实现 schema 验证器，至少检查 8 项：key、shape、dtype、有限值、范围、时间单调、图像通道、指令非空。
- 打印一个训练 batch 的转换前后 shape，并解释每次 transpose/stack。

## 6. 输入、预期输出与验证

输入：两条合法记录；五条分别含错 shape、`float64` 图像、动作越界、`NaN`、时间倒退的记录。

预期输出：合法记录通过；五条非法记录分别给出可定位错误消息，例如 `action[0]=0.2 exceeds 0.05 m`。这是预期结果。

验证：测试应同时检查“会失败”和“失败原因正确”；随机抽 32 步堆叠后断言 batch shape；反归一化后与原动作 `allclose`。

## 7. 固定提交格式

```markdown
# W5D3 提交
- 实际用时：
## Schema 字段表
## 状态/动作逐维定义
## Batch 形状流
## 验证器测试矩阵
## 命令与完整日志
## 自测答案
```

## 8. 评分与验收

- schema 六要素齐全：25 分。
- 逐维动作、坐标系与单位明确：20 分。
- 验证器覆盖至少 8 项：25 分。
- batch/归一化往返验证：20 分。
- 日志与自测：10 分。

总分 100，80 分通过；动作坐标系、单位或时间语义任一缺失时不通过。

## 9. 常见错误与排查

| 错误 | 排查 |
|---|---|
| HWC/CHW 混淆 | 在环境边界和模型边界分别断言 |
| 四元数顺序不明 | 字段名直接写 `qx,qy,qz,qw` 或 `qw,qx,qy,qz` |
| 用全数据统计归一化 | 先 split，再只计算 train 统计 |
| gripper 同时用 0/1 与 -1/1 | 在 adapter 处只保留一个规范 |

## 10. 时间不足时的最低完成线

完成 CPU Mock 的 state/action 逐维表，以及 shape、范围、有限值、时间单调四类自动断言。

## 11. 可选提高

用 JSON Schema/Pydantic 表达元数据；增加 schema 版本迁移规则；统计每维动作分位数并识别饱和。

## 12. 完成后的自测题

1. 为什么 shape 相同仍可能语义不兼容？
2. 为什么 normalization statistics 不能用测试集？
3. 图像保存 HWC、训练使用 CHW 的边界在哪里？
4. 动作范围检查为什么要逐维而非只看范数？

[上一课：Day 2](day-02.md) · [本周首页](README.md) · [下一课：Day 4](day-04.md)
