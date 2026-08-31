# 第 6 周 Day 3：视觉特征与机器人状态融合

[填写本日 Notebook 作业](../notebooks/week-06/day-03.ipynb)

[上一课：Day 2](day-02.md) · [本周首页](README.md) · [下一课：Day 4](day-04.md)

## 1. 今日目标与预计时间

预计 120 分钟。实现 `image + state → action` 融合模型，并用扰动实验证明输出同时依赖视觉和机器人状态。

## 2. 这在 VLA 中的作用

图像告诉策略目标与场景，proprioception 告诉策略机械臂自身位置。只依赖任一模态都可能在训练数据相关性下得到低 loss，却无法在目标或初始姿态变化时闭环控制。

## 3. 核心概念、公式与张量

最小 late-fusion：

\[
z_i=f_{img}(I_t)\in\mathbb{R}^{B\times D_i},\quad
z_s=f_s(s_t)\in\mathbb{R}^{B\times D_s},
\]

\[
\hat a_t=f_a([z_i;z_s])\in\mathbb{R}^{B\times d_a}.
\]

示例：`image [B,3,64,64]→[B,128]`，`state [B,8]→[B,32]`，拼接 `[B,160]`，输出 `[B,7]`。state 的 normalization 统计只取 train。

## 4. 分步学习

1. 为 state 写逐维 normalization 和反变换测试。
2. 编写 state MLP 与视觉 encoder，并在 forward 起点断言 batch 一致。
3. 拼接后用 MLP 输出归一化动作；保存 action denormalize 函数供 rollout 共用。
4. 固定 state、替换图像，观察输出差；固定图像、替换 state，再观察输出差。
5. 增加 image-zero 与 state-zero 两个消融，记录输出和梯度变化。

## 5. 必做作业

- 实现 `VisualStatePolicy`，输出 shape 与 schema 一致。
- forward/backward 至少各一次，并打印各分支梯度范数。
- 做四对配对样本敏感性测试，证明两种输入变化会影响动作；若不影响，定位原因而非强行判通过。

## 6. 输入、预期输出与验证

输入：batch `B=4`；两幅目标位置不同的图像；两个末端位置不同的 state；动作维 `d_a` 取你的 schema。

预期输出：`pred [4,d_a]` 有限；两个分支有非零梯度；图像或 state 替换使至少一个相关动作维变化。这里只描述预期行为，阈值应由实际初始模型/训练状态说明。

验证：加入 batch mismatch 负向测试；归一化再反变换 `allclose`；禁止仅用完全随机未训练网络的输出差宣称学到语义，敏感性测试需区分“结构连通”和“训练后有意义依赖”。

## 7. 固定提交格式

```markdown
# W6D3 提交
- 实际用时：
## 融合结构与 shape 流
## State/action normalization
## Forward/backward 日志
## 模态敏感性与消融表
## 负向测试
## 自测答案
```

## 8. 评分与验收

- 融合结构和 shape 正确：20 分。
- normalization 往返可靠：20 分。
- 两分支梯度证据：20 分。
- 模态敏感性/消融解释严谨：25 分。
- 负向测试与自测：15 分。

总分 100，80 分通过；batch 静默广播、动作未反归一化或把随机敏感性当语义 grounding 时不通过。

## 9. 常见错误与排查

| 错误 | 排查 |
|---|---|
| state 被图像 batch 错配 | 每条样本带 episode_id/step_index 并在 collate 检查 |
| 图像分支梯度为零 | 检查 detach、no_grad、冻结配置和 concat 输入 |
| 输出动作尺度异常 | forward 输出规范化动作，环境前统一反变换/clip |
| 模型忽略图像 | 检查数据中 state 是否已泄漏目标信息 |

## 10. 时间不足时的最低完成线

使用 synthetic batch 和 tiny CNN，完成 shape、两分支梯度、normalization 往返及 batch mismatch 四项测试。

## 11. 可选提高

比较 concat、FiLM 与 gated fusion；保留 spatial tokens 做 attention pooling；估算各融合结构参数量。

## 12. 完成后的自测题

1. 为什么 state 可能泄漏目标信息？
2. 模态敏感性与真正语义依赖有什么区别？
3. 反归一化应在哪个系统边界进行？
4. early/late fusion 各有什么取舍？

[上一课：Day 2](day-02.md) · [本周首页](README.md) · [下一课：Day 4](day-04.md)
