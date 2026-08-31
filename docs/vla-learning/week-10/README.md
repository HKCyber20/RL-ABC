# 第 10 周：OpenVLA：推理、动作解码与低资源适配

[打开本周 Notebook 作业](../notebooks/week-10/README.md)

[← 第 9 周](../week-09/README.md) · [课程总入口](../README.md) · [第 11 周 →](../week-11/README.md)

## 本周核心能力与总 Goal 中的作用

**核心能力：**能解释 OpenVLA 从图像与指令到动作的端到端路径，审查推理接口与动作反归一化，设计自定义数据 adapter，并在 LoRA、OFT 与全参数微调之间做有证据的资源决策。

本周把第 9 周的数据契约连接到一个开源 VLA。最终 Goal 不要求从零训练 7B 模型，但要求能够运行推理/小规模适配，或在资源不足时提供可复核的 mock dry-run、配置审计和接口测试证据。

## 前置知识

- 第 9 周的 RLDS schema、动作契约与 inspector 已通过。
- 理解 processor、视觉 token、语言 token、动作 token/连续动作头的基本职责。
- 会读取配置、估算参数/激活/优化器内存，并用 mock 对象做接口测试。

## 推荐一手资料

- [OpenVLA 官方仓库](https://github.com/openvla/openvla)

官方接口、依赖和训练配方会演进；任何真实命令都要绑定实际使用的 commit、模型 revision 与环境锁，不能只依赖课程中的抽象接口。

## 7 天导航

| 日程 | 课程 | 当天可验证结果 |
|---|---|---|
| Day 1 | [OpenVLA 架构与端到端数据流](day-01.md) | 识别 OpenVLA 中视觉编码器、投影/融合、语言模型、action tokenizer/预测接口的职责。 |
| Day 2 | [环境与算力规划：先做 go/no-go 决策](day-02.md) | 列出真实推理/微调的环境、下载、磁盘和显存成本。 |
| Day 3 | [推理接口：processor、prompt 与 predict_action](day-03.md) | 定义可测试的推理函数签名与异常行为。 |
| Day 4 | [图像与语言预处理契约](day-04.md) | 验证 RGB、尺寸、值域、批维和 prompt 模板。 |
| Day 5 | [动作反归一化与安全边界](day-05.md) | 实现 normalized action 到物理动作的可逆映射。 |
| Day 6 | [自定义 dataset adapter 与样例 batch](day-06.md) | 把自定义 episode 映射为 OpenVLA 需要的统一样本。 |
| Day 7 | [LoRA、OFT、全参微调取舍与周项目](day-07.md) | 基于目标、资源和部署频率选择适配方案。 |

## 本周代码与实验产物

- openvla_architecture.md：稳定模块、版本相关接口和张量/数据流图。
- compute_plan.md 与 environment_lock.md：下载、磁盘、显存、运行时间及低资源替代。
- mock_openvla_inference.py：processor/model/predict_action 接口桩与测试。
- action_stats.json、unnormalize_action.py 与 round-trip 日志。
- custom_dataset_adapter.py、样例 batch 与 schema 测试。
- adaptation_decision.md、dry-run 配置和 w10_week_project.md。

## 必做任务

- 所有运行路径显式携带 dataset/unnorm key，缺失时立即失败。
- 推理前处理、动作解码与 controller contract 各有独立测试。
- 真实下载、安装或 GPU 训练只作为可选路径，执行前记录成本并获得用户授权。
- 不能把 mock 输出冒充模型语义结果，也不能把静态检查冒充 checkpoint 推理。

## 可选任务

- 在资源允许并获得授权后，按所用官方版本运行最小 checkpoint 推理。
- 阅读当前 OpenVLA/OFT 官方说明并记录 commit 或发布日期。
- 对比离散 action tokenizer、FAST 与 OFT 连续 action chunk 的接口差异。

## 时间不足时的最低完成线

全程 CPU/无网络：使用 mock processor、mock model、2 条自建样本与小型 action stats 完成推理接口 dry-run、反归一化 round-trip、dataset adapter 单测和适配决策表。

## 周末概念测验

1. 画出 PIL image + instruction → processor → model → normalized action → unnormalize → controller。
2. 解释为什么同一 normalized action 在不同 dataset key 下可能对应不同物理动作。
3. 列出真实推理前至少 8 个环境/资源检查项。
4. 比较 LoRA、OFT 和全参数微调的训练对象、推理形态、资源与适用目标。

闭卷 80 分通过。unnorm key、动作单位/frame 或 mock/real 证据等级答错属于硬门槛，须补救后复验。

## 周项目

实现一个不依赖真实 checkpoint 的 OpenVLA 接入模拟器：两条样本经 adapter、processor contract、mock predict_action、dataset-specific 反归一化与 controller safety check，输出结构化 trace；再提供真实推理/微调的资源计划与 go/no-go 决策。

## 本周通过标准

- 7 个日课均达到 80/100，且动作反归一化硬门槛全部通过。
- mock pipeline 能对合法样本生成完整 trace，并拒绝错误图像模式、空指令、未知 unnorm key、动作维度错和越界部署动作。
- 自定义 adapter 的输入/输出、时间对齐、语言字段和 action stats 来源可追踪。
- 能够解释 OpenVLA 官方接口可能随版本变化，真实命令必须绑定所用 commit/依赖；没有伪造任何实测结果。

周进度仅在逐项证据通过后勾选；静态、mock 与真实运行分别记录，不能互相冒充。

---

[← 第 9 周](../week-09/README.md) · [课程总入口](../README.md) · [第 11 周 →](../week-11/README.md)
