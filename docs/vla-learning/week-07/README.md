# 第 7 周：语言条件策略与跨模态对齐

[打开本周 Notebook 作业](../notebooks/week-07/README.md)

[返回课程总览](../README.md)

## 本周核心能力

本周在已验证的视觉 BC 上加入语言：构建 tokenizer/text encoder，设计可控指令与组合切分，实现 concat/FiLM 基线和 cross-attention，用反事实探针验证语言是否真的改变目标选择，并评估未见属性组合。它对应总 Goal 中 `图像 + 语言 + 机器人状态 → 动作` 的完整输入侧。

## 前置知识

- 第 6 周视觉 + state 策略能完成 forward/backward 和闭环评估。
- 理解 token、embedding、padding mask、Q/K/V 和 cross-attention。
- 具备按 episode 切分数据、保存词表与 checkpoint 的能力。
- 任务数据中可识别源物体、目标容器与指令之间的对应关系。

## 7 天导航

| 天 | 主题 | 当天可验证产物 |
|---|---|---|
| [Day 1](day-01.md) | Tokenizer 与文本编码器 | 词表、mask 与 encoder 测试 |
| [Day 2](day-02.md) | 指令设计与数据审计 | 指令 grammar、覆盖矩阵与 split |
| [Day 3](day-03.md) | 融合基线 | `image+text+state` policy 与反事实测试 |
| [Day 4](day-04.md) | Cross-attention | 形状/掩码/权重检查与动作头 |
| [Day 5](day-05.md) | Grounding 探针 | 指令/场景交换实验与证据表 |
| [Day 6](day-06.md) | 未见组合评估 | compositional split 与分组指标 |
| [Day 7](day-07.md) | 周项目 | 语言条件策略、rollout 与失败分析 |

## 本周代码与实验产物

- 固定版本的 tokenizer 词表、规范化规则和 text encoder。
- 指令 grammar、组合覆盖矩阵、train/val/test 清单。
- concat/FiLM 或 cross-attention 的语言条件策略。
- counterfactual grounding 探针与 seen/unseen-combination 指标。
- `language_policy_report.md` 及逐 rollout 原始记录。

课程没有预先运行这些模型；出现的行为均为预期检查目标，实际结果必须由日志支持。

## 必做与可选任务

必做：7 日核心任务；同一视觉场景切换指令；至少一组指令/图像反事实探针；评估 seen 与未见组合；保留无语言或固定语言 baseline。

可选：加入预训练文本 encoder、同义改写、对比学习或多语言指令。预训练权重需要下载时应先评估成本，本周可完全用本地小词表和 tiny Transformer/GRU 在 CPU 完成。

## 最低完成线

使用结构化 4–12 个组合、字符或词级 tokenizer、tiny text encoder、synthetic 双目标场景。训练数据最多数百帧；完成同图换指令、同指令换目标位置、指令 token 打乱三类测试，并在至少 10 次 seen 与 10 次 unseen-combination Mock rollout 上报告结果。

## 周末测验

解释 padding mask 的方向；未知词与未见组合的差别；concat 与 cross-attention 的信息流；为什么 attention 权重不是 grounding 的充分证据；如何构造“属性都见过但组合没见过”的 split；为什么同一场景换指令是关键测试。

## 周项目

训练 `RGB + instruction + state → action` 策略，使同一场景中不同指令选择不同目标。报告 seen/unseen-combination 成功率，并与固定指令或移除语言 baseline 对照；分析至少 3 个语言绑定失败。

## 通过标准

- 7 个日课均达到通过线，Day 7 至少 80/100。
- tokenizer、词表、padding/truncation 规则随 checkpoint 保存。
- 数据切分无 episode/模板泄漏，组合覆盖可审计。
- 至少一种反事实测试证明动作或目标选择随语义变化，而非仅随长度/模板变化。
- seen 与 unseen-combination 分开报告成功数/总数和失败类型。
