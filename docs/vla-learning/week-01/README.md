# 第 1 周：Transformer 与 VLA 心智模型

[打开本周 Notebook 作业](../notebooks/week-01/README.md)

[← 返回课程总览](../README.md)

## 本周核心能力

本周建立贯穿全课程的计算图：把图像、语言和机器人状态表示成张量，通过注意力融合后预测动作，并把动作放回环境形成闭环。它对应总 Goal 中“能够解释 VLA、构造图像 + 语言 + 状态 → 动作数据”的理论底座。

完成后应能独立说明 token、embedding、位置、Q/K/V、mask、self-attention 与 cross-attention；并实现一个输出 `[B, 7]` 动作的最小融合网络，完成 forward、loss、backward。

## 前置知识

- Python 基本语法、函数与列表；
- 线性代数中的向量、矩阵乘法；
- 知道神经网络可通过损失和梯度更新参数即可；
- 有 PyTorch 可做完整实验；没有 GPU 也可用 CPU。若暂时没有 PyTorch，Day 1–5 可用纸笔或 NumPy，Day 6 使用给出的形状推演作为最低完成线。

## 7 天导航

1. [Day 1：VLA 闭环与四类变量](day-01.md)
2. [Day 2：Token、Embedding 与位置](day-02.md)
3. [Day 3：Q/K/V 与 Scaled Dot-Product Attention](day-03.md)
4. [Day 4：Causal Mask 与 Multi-Head Attention](day-04.md)
5. [Day 5：Cross-Attention 视觉语言融合](day-05.md)
6. [Day 6：Toy Fusion Network 的前向与反向](day-06.md)
7. [Day 7：周测与最小 VLA 周项目](day-07.md)

## 本周代码与实验产物

- 一份 VLA 闭环变量说明和张量契约；
- token/patch 数量与 embedding 形状推演；
- 手写单头 scaled dot-product attention；
- causal mask 与多头形状检查；
- 文本查询图像 patch 的 cross-attention 实验；
- `image_tokens + text_tokens + state -> action` toy 网络及 forward/backward 日志；
- 周项目报告，包含至少一项错误注入与解释。

## 必做与可选任务

必做是 7 天中的“最低完成线”以及 Day 7 周项目。可选任务用于加深理解，不影响通过。所有矩阵规模都很小，CPU 足够；不要下载预训练模型或大型数据。若无法运行代码，必须手算一个样例并提交完整中间矩阵，后续具备环境时再补做 backward 才能获得本周完整通过。

## 最低完成线

- 能画出 `o_t, l, s_t -> policy -> a_t -> env -> o_{t+1}, s_{t+1}`；
- 手算并解释一次 `softmax(QK^T/sqrt(d_k))V`；
- 正确写出 self-attention 与 cross-attention 的形状；
- toy 网络至少通过前向形状断言；
- 解释为什么低离线 loss 不保证高闭环成功率。

## 周末测验、周项目与通过标准

Day 7 包含 30 分概念测验与 70 分周项目。周项目要求融合随机图像 token、文本 token 和机器人状态，输出 7 维连续动作；检查注意力权重、输出形状、有限 loss 和至少一个非空梯度。总分达到 80/100，且“张量形状、跨模态方向、闭环解释”三个关键项均合格才通过。未通过时只补对应证据，不重做已通过部分。

## 进度规则

课程文件预先存在不代表完成。只有提交对应证据并按 rubric 验收后，才可在 `../PROGRESS.md` 勾选。通过本周后进入[第 2 周](../week-02/README.md)。

[← 返回课程总览](../README.md) · [开始 Day 1 →](day-01.md)
