# 第 7 周 Day 3：语言、视觉与状态的融合基线

[填写本日 Notebook 作业](../notebooks/week-07/day-03.ipynb)

[上一课：Day 2](day-02.md) · [本周首页](README.md) · [下一课：Day 4](day-04.md)

## 1. 今日目标与预计时间

预计 120 分钟。实现一个清晰的 concat 或 FiLM 语言条件基线，并通过同图换指令、同指令换图和语言消融确认三模态通路连通。

## 2. 这在 VLA 中的作用

简单融合是 cross-attention 的必要 baseline。若 concat 都无法使用语言，复杂注意力往往只会增加调试难度；反之，简单结构成功可说明任务不一定需要更重模型。

## 3. 核心概念、公式与张量

concat baseline：

\[
\hat a=f([f_I(I_t); f_L(l); f_S(s_t)]).
\]

示例 shape：`z_img [B,128]`、`z_text [B,64]`、`z_state [B,32]`，融合 `[B,224]`，动作 `[B,d_a]`。

FiLM 可让文本调制视觉：`z'_img=γ(z_text)⊙z_img+β(z_text)`。两者都应有无语言 baseline（固定零向量或固定指令），但零向量必须在训练/评估一致。

## 4. 分步学习

1. 载入第 6 周 encoder 和 Day 1 text encoder，统一 batch/device/dtype。
2. 实现 concat baseline；可选再加 FiLM，不要同时改训练协议。
3. 用 paired batch：同一图/state 配两条不同目标指令。
4. forward/backward，记录 image/text/state 三分支梯度。
5. 训练一个极小 synthetic batch，验证指令改变时预期动作方向改变。
6. 做语言置零、token 打乱、同长度错指令三种消融。

## 5. 必做作业

- 实现 `LanguageConditionedPolicy`，列出全程 shape。
- 在同一场景的两条冲突指令上做成对测试；未训练模型只证明结构连通，训练后再判断语义。
- 输出三分支梯度及三种语言消融对动作的影响。

## 6. 输入、预期输出与验证

输入：含红/蓝两个候选目标的同一图像、同一 state、两条分别指向红/蓝的指令；batch 至少 4。

预期输出：模型输出 `[B,d_a]`；三分支可训练时均有有限梯度；短训练后冲突指令产生朝不同目标的动作趋势。实际是否达到必须由数据与日志证明。

验证：交换 batch 中文本顺序，输出也应按样本重排；同长度错误指令排除长度捷径；state/动作 normalization 与第 6 周完全复用。

## 7. 固定提交格式

```markdown
# W7D3 提交
- 实际用时：
## 融合结构与 shape 表
## Paired batch 构造
## Forward/backward 日志
## 同图换指令结果
## 三种语言消融
## 自测答案
```

## 8. 评分与验收

- 三模态 shape 和融合实现：20 分。
- 三分支梯度与 batch 对齐：20 分。
- 同图冲突指令测试：25 分。
- 三种语言消融：20 分。
- 结论边界与自测：15 分。

总分 100，80 分通过；文本错配、只用不同长度指令或用未训练随机输出声称 grounding 时不通过。

## 9. 常见错误与排查

| 错误 | 排查 |
|---|---|
| 文本分支梯度为零 | 检查 detach、冻结、concat 维和 optimizer |
| 模型忽略文本 | 检查任务是否可由 state/图像唯一决定及训练组合平衡 |
| FiLM 数值爆炸 | 限制/初始化 gamma、beta，监控特征范数 |
| paired batch 实际场景不同 | 固定同一 image/state，仅换 token ids/mask |

## 10. 时间不足时的最低完成线

只做 concat + CPU tiny encoders，通过 shape、三分支梯度、同图换指令和同长度错指令四项测试。

## 11. 可选提高

比较 concat 与 FiLM 参数量/收敛；加入 modality dropout；画文本 embedding 的简单相似度矩阵。

## 12. 完成后的自测题

1. 为什么 concat baseline 必不可少？
2. 同长度错指令能排除哪种捷径？
3. 分支有梯度能否证明模型正确理解语言？
4. FiLM 与 concat 的信息交互方式有何不同？

[上一课：Day 2](day-02.md) · [本周首页](README.md) · [下一课：Day 4](day-04.md)
