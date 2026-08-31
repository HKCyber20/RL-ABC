# 第 6 周：视觉行为克隆

[打开本周 Notebook 作业](../notebooks/week-06/README.md)

[返回课程总览](../README.md)

## 本周核心能力

本周把第 5 周的可回放图像轨迹变成 `image + state → action` 策略：建立视觉预处理与编码器，比较冻结和微调，完成多模态融合、训练、增强、离线评估和闭环 rollout。它直接支撑总 Goal 中的“训练并闭环评估语言条件机器人策略”；先把视觉控制做对，下一周加入语言时才能判断问题来自视觉还是语言绑定。

## 前置知识

- 第 5 周数据通过 schema、时间对齐和回放检查。
- 熟悉 PyTorch Dataset/DataLoader、反向传播和 checkpoint 基础。
- 理解行为克隆损失与闭环分布偏移。
- 能说明动作每一维的单位、坐标系、范围及归一化。

## 7 天导航

| 天 | 主题 | 当天可验证产物 |
|---|---|---|
| [Day 1](day-01.md) | 视觉预处理与编码器 | 预处理单测 + encoder shape/梯度日志 |
| [Day 2](day-02.md) | 冻结与微调 | trainable 参数审计 + 两种模式 smoke test |
| [Day 3](day-03.md) | 视觉与状态融合 | 融合模型 + 模态敏感性测试 |
| [Day 4](day-04.md) | BC 训练流水线 | 小批过拟合 + checkpoint 可复现 |
| [Day 5](day-05.md) | 数据增强 | 增强可视化 + 标签一致性测试 |
| [Day 6](day-06.md) | 离线指标与 rollout | 指标对照与分布偏移实验 |
| [Day 7](day-07.md) | 周项目 | 视觉 BC 与 state-only 对照报告 |

## 本周代码与实验产物

- 数据加载和视觉预处理检查脚本。
- `VisualEncoder`、`VisualStatePolicy` 与 state-only baseline。
- 训练配置、随机种子、归一化统计和可恢复 checkpoint。
- 训练/验证曲线、逐动作维指标、至少一组闭环成功率。
- `visual_bc_report.md`，包含失败案例和离线—在线差异。

文档中的数值只表示预期形态，不表示本机已运行；提交时必须提供真实命令与日志。

## 必做与可选任务

必做：7 天核心任务；以 episode 为单位划分 train/val/test；训练视觉 + state 策略与 state-only baseline；报告离线误差和 rollout 成功率。

可选：使用预训练视觉编码器、多视角融合、混合精度或更大图像。任何下载和 GPU 训练都不是必需条件。

## 最低完成线

使用第 5 周 synthetic RGB 或程序生成 `64×64` 图像、CPU tiny CNN、最多数百步数据；完成 32 样本过拟合、按 episode 切分、视觉 + state 与 state-only 对照、每个模型至少 10 次 Mock rollout。不能用单帧随机切分替代 episode 切分。

## 周末测验

解释：预处理和增强的区别；冻结 encoder 时如何验证；为何验证 MSE 不能代替成功率；episode split 如何防泄漏；空间增强为何可能要求同步变换动作；如何判断模型忽略视觉。

## 周项目

训练 `rgb_t + state_t → action_t` 的视觉 BC，并与 `state_t → action_t` 对照。在相同数据、seed、动作归一化和评估 episode 上给出训练曲线、逐维误差及闭环成功率；至少分析 3 个失败 episode。

## 通过标准

- Day 1–7 均达到各自通过线，Day 7 至少 80/100。
- encoder/fusion 的 shape 断言和梯度审计通过。
- 数据按 episode 切分，归一化统计仅来自 train。
- checkpoint 重载后固定输入输出一致。
- 同时报离线指标与 rollout 成功率，并能用证据说明两者差异。
