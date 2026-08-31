# 第 12 周：可复现 VLA 最终项目与答辩

[打开本周 Notebook 作业](../notebooks/week-12/README.md)

[← 第 11 周](../week-11/README.md) · [课程总入口](../README.md)

## 本周核心能力与总 Goal 中的作用

**核心能力：**冻结任务 spec 与可复现环境，完成数据卡、最终训练/复跑、闭环泛化评估、失败分析、demo 与报告，并通过从原始数据到结果表的端到端复现和口头答辩。

这是 12 周总 Goal 的验收周。最终交付必须证明你能构造图像+语言+机器人状态→动作的数据，训练并闭环评估策略，分析泛化与失败，并理解和运行 OpenVLA 推理/适配流程或提供严格标注的低资源 dry-run 证据。

## 前置知识

- 前 11 周核心硬门槛已通过，尤其是动作 frame/单位、时间对齐、评估 split 与失败 taxonomy。
- 已有一个能执行的语言条件策略和仿真环境；低资源可使用自建 2D/网格 RGB 闭环环境。
- 已有 OpenVLA mock/真实接口、action stats 与 adapter 证据。

## 7 天导航

| 日程 | 课程 | 当天可验证结果 |
|---|---|---|
| Day 1 | [冻结最终项目 spec 与验收合同](day-01.md) | 把任务范围、输入、动作、控制频率和成功判据冻结为可测试合同。 |
| Day 2 | [配置、随机种子与环境复现](day-02.md) | 建立单一配置入口与 seed 传播表。 |
| Day 3 | [数据卡、manifest 与质量签名](day-03.md) | 完成最终数据卡与样本级 manifest。 |
| Day 4 | [最终训练或低资源确定性复跑](day-04.md) | 从冻结 config 训练/加载最终策略。 |
| Day 5 | [最终闭环评估与结果冻结](day-05.md) | 执行 ID 与至少两类 OOD 多 seed 评估。 |
| Day 6 | [Demo、最终报告与一键复现说明](day-06.md) | 制作不挑样本的代表性 demo。 |
| Day 7 | [答辩、总验收与后续路线](day-07.md) | 完成 12 周总 Goal 的证据审计。 |

## 本周代码与实验产物

- FINAL_SPEC.md：冻结任务、输入输出、动作、成功判据和范围。
- configs/ 与 ENVIRONMENT.md：版本、seed、命令和资源档位。
- DATA_CARD.md、dataset manifest/hash 与质量报告。
- 最终 checkpoint/确定性基线或低资源训练产物及 train/eval 日志。
- FINAL_RESULTS.md：闭环 ID/OOD 成功率、CI、失败 taxonomy 与消融。
- DEMO.md/视频或帧序列、FINAL_REPORT.md、REPRODUCE.md 和答辩记录。

## 必做任务

- 最终结果表必须来自真实执行的闭环仿真；可用轻量自建环境，但不能用手写 mock outcome 代替。
- 原始数据、split、统计、训练配置、checkpoint、评估 manifest 和报告之间均有 hash/id 追踪。
- 至少 ID、未见位置和未见组合/背景中的两类 OOD；报告失败分布与代表案例。
- OpenVLA 至少提供真实推理/processor 证据，或完整 CPU mock dry-run 并明确其不能证明语义性能。

## 可选任务

- 在授权和资源允许时运行真实 OpenVLA checkpoint 或 LoRA/OFT 小规模适配。
- 容器化、CI 一键烟雾测试或发布匿名复现实验包。
- 制作 5 分钟演示视频和一页海报。

## 时间不足时的最低完成线

使用 CPU 轻量 2D/网格 RGB 仿真：至少两条语言指令、机器人状态、7 维或明确降维的动作契约，训练/加载一个小型 BC 策略，真实闭环运行多 seed，完成 ID+两类 OOD、失败分析；OpenVLA 用第 10 周 mock dry-run 与 adapter/unnorm 证据。

## 周末与最终概念测验

1. 从图像采集到 controller 下发，逐层说出 shape、单位、frame 和检查。
2. 为什么最终结果必须从 episode manifest 重算，而不能从训练日志抄？
3. 如何证明未见组合测试没有数据泄漏？
4. OpenVLA mock dry-run与真实 checkpoint 推理分别证明什么？

闭卷至少 80 分；shape/frame、结果追踪、split 泄漏和 mock/real 边界为硬门槛。

## 周项目

从干净环境或明确记录的现有环境，按 REPRODUCE.md 完成数据检查、训练/加载、闭环评估和报告生成；另一人应能在不猜路径、seed 或统计 key 的情况下复跑最小实验。

## 本周与 12 周最终通过标准

- 可复现仿真 VLA：存在实际可运行环境和策略，能接收图像、语言、机器人状态并输出动作，在闭环中完成至少一个语言条件任务。
- 数据：schema、时间对齐、split、动作单位/frame、统计 key、数据卡和质量审计齐全。
- 闭环评估：多 seed、成功率分母、CI、ID 与至少两类 OOD、episode 级追踪和失败 taxonomy 齐全。
- 泛化与失败：未见位置及未见组合/背景至少两轴有证据，包含代表失败、根因假设和最小消融。
- OpenVLA：能准确解释架构、processor、adapter、动作反归一化和 LoRA/OFT/全参取舍；提供真实运行证据或清楚标注的 CPU/mock dry-run trace。
- 复现：从配置到主要结果表至少复跑一次；环境限制导致未跑部分必须标明，且最终核心闭环不能只停留在预期。
- 答辩总分至少 80/100，任何数据伪造、mock/real 混淆、动作语义缺失或结果不可追踪均直接不通过。

只有教练基于证据完成验收后才能更新最终进度；文件存在、口头声称完成或只有预期输出均不构成通过。

---

[← 第 11 周](../week-11/README.md) · [课程总入口](../README.md)
