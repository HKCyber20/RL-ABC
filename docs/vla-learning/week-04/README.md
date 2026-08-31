# 第 4 周：时序建模与 Action Chunk

[打开本周 Notebook 作业](../notebooks/week-04/README.md)

[← 返回课程总览](../README.md)

## 本周核心能力

本周从单步 BC 扩展到时序策略：对齐图像/状态/动作时间戳，构造历史窗口和变长序列，使用 padding mask，预测未来 action chunk，并评估控制频率、推理延迟和动作重规划。它为总 Goal 中的视觉 VLA 数据、Transformer policy 和闭环稳定评估奠定直接基础。

## 前置知识

- 第 3 周可复跑的低维 BC 数据与 rollout；
- 第 1 周 attention/mask 和 batch/sequence 形状；
- 第 2 周动作单位、坐标系和控制频率；
- PyTorch CPU 足够；没有框架可用 NumPy/纸笔完成窗口、mask、chunk 与调度实验。

## 7 天导航

1. [Day 1：图像、状态、动作的时间对齐](day-01.md)
2. [Day 2：历史帧、部分可观测性与堆叠](day-02.md)
3. [Day 3：Sequence Dataset、Padding 与 Mask](day-03.md)
4. [Day 4：Action Chunk 标签与执行策略](day-04.md)
5. [Day 5：最小 Transformer Policy](day-05.md)
6. [Day 6：控制频率、推理延迟与重规划](day-06.md)
7. [Day 7：周项目——单步与 Action Chunk 对比](day-07.md)

## 本周代码与实验产物

- 多传感器 timestamp 对齐器及偏移一帧反例；
- 历史窗口 dataset 与速度不可观测实验；
- padding/causal/key-padding mask 的变长序列 batch；
- action chunk 标签、valid mask 和重叠执行调度器；
- 输入历史并输出 `[B,H_a,d_a]` 的小型 Transformer policy；
- 频率/延迟/重规划消融表；
- 单步对 action chunk 的同协议周项目报告。

## 必做与可选任务

必做聚焦时间索引、valid mask、无跨 episode、闭环调度和公平比较。可选包括 temporal ensembling 与缓存。低资源方案使用第 3 周二维环境、短序列、小模型；若 Transformer 训练耗时，则只训练线性/MLP chunk 基线并对 Transformer 完成 forward/backward 形状验证。

## 最低完成线

- 画清 `obs_t,state_t -> action_t -> obs_{t+1}` 并检测一帧偏移；
- 构造历史窗口和变长 batch，mask 语义正确；
- chunk 标签不跨 episode，尾部用 valid mask 排除；
- Transformer/chunk 模型输出形状与 loss mask 正确；
- 在同一 20+ seeds 上比较单步与 chunk，报告延迟、成功率和失败类型。

## 周末测验、周项目与通过标准

Day 7 周测 25 分、项目 75 分。80/100 通过；时间对齐、padding mask、chunk 边界、执行频率四项为门槛。对比不要求 chunk 一定更好，要求配置公平、结果真实、失败可解释。

## 进度规则

课程预创建不代表完成。只有 rubric 证据通过才更新 `../PROGRESS.md`。完成后进入[第 5 周](../week-05/README.md)，开始视觉操作仿真与数据采集。

[← 返回课程总览](../README.md) · [开始 Day 1 →](day-01.md)
