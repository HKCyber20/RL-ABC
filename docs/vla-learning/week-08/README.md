# 第 8 周：最小 VLA 端到端集成

[打开本周 Notebook 作业](../notebooks/week-08/README.md)

[返回课程总览](../README.md)

## 本周核心能力

本周将视觉、语言、机器人状态和动作整合为一个可复现的最小 VLA：冻结总体接口，比较连续动作回归与离散动作 token，打通训练流水线，实施闭环推理与 action chunk，建立系统级调试顺序，并在训练分布上完成正式周项目。它是总 Goal 的第一个端到端里程碑，也是后续阅读 RT-2/OpenVLA 的实践参照系。

## 前置知识

- 第 5 周数据 schema、回放和 success checker 可用。
- 第 6 周视觉 BC 能训练、重载并闭环评估。
- 第 7 周 tokenizer、语言条件策略和 grounding 探针通过。
- 能明确动作坐标系、单位、控制频率、归一化和安全边界。

## 7 天导航

| 天 | 主题 | 当天可验证产物 |
|---|---|---|
| [Day 1](day-01.md) | 总体架构与接口冻结 | 端到端数据流、模块契约、smoke test |
| [Day 2](day-02.md) | 连续动作回归 | 连续 action head、loss 与反变换测试 |
| [Day 3](day-03.md) | 离散动作 token | 分桶/解码器、量化误差与对照 |
| [Day 4](day-04.md) | 训练流水线 | config 驱动训练、checkpoint、dry run |
| [Day 5](day-05.md) | 闭环推理与 action chunk | receding-horizon rollout 与安全日志 |
| [Day 6](day-06.md) | 系统调试 | 分层诊断矩阵与故障注入 |
| [Day 7](day-07.md) | In-distribution 周项目 | 20 次 rollout 或低资源替代、报告与复跑 |

## 本周代码与实验产物

- 版本化 `VLAInput/VLAOutput` 契约与端到端 shape 测试。
- 连续动作 head，以及教学用离散 token 编解码器。
- 配置、split、statistics、checkpoint 和训练/评估入口。
- receding-horizon/action-chunk rollout、动作安全检查与逐步日志。
- ID 评估报告、成功率、阶段指标和至少 5 个失败案例候选（实际失败不足时提交全部失败，不虚构）。

文中的成功行为与阈值均是验收目标或示例，不代表已在本机运行。

## 必做与可选任务

必做：7 日核心任务；连续动作主线；离散动作 token 做可验证小实验；至少完成训练 dry run、小批过拟合、checkpoint 重载、闭环 rollout、故障注入和 ID 报告。

可选：真正训练离散 token policy、加入时间序列 Transformer、多相机、action chunk temporal ensembling。无需下载大模型，也不要求 GPU。

## 最低完成线

使用 CPU、synthetic RGB、tiny encoders 和第 5 周 Mock 环境；连续动作 `d_a=3` 或你的规范动作维，chunk `H=2–4`；完成 10 次 ID autonomous rollout。主线正式验收推荐至少 20 次。所有低资源结果也必须有逐 episode 记录、成功判定和失败分类。

## 周末测验

解释：VLA 与视觉 BC 的新增接口；连续/离散动作的误差来源；为什么量化误差下界必须测；teacher-forced loss 与闭环差异；action chunk 的预测/执行时域；receding horizon 的意义；调试为何先数据后模型。

## 周项目

交付最小端到端原型：

```text
RGB + instruction token/mask + robot state
                    ↓
            multimodal policy
                    ↓
           continuous action chunk
                    ↓
       safety transform + environment
                    ↓
               next observation
```

在 ID 场景上固定至少 20 个 rollout seeds；低资源最低 10 个。报告成功数/总数、阶段到达率、平均长度、越界修正数及失败分类。

## 通过标准

- Day 1–7 各达到通过线，Day 7 至少 80/100。
- 一条真实/Mock episode 从磁盘数据到训练、checkpoint、闭环评估可追溯。
- 连续动作 normalization/denormalization 和离散动作 encode/decode 有往返测试。
- action chunk 的标签、预测、执行索引无歧义，安全边界有日志。
- ID rollout 有固定 seeds 和逐 episode 证据，结果可从原始记录重算。
