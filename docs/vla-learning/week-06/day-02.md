# 第 6 周 Day 2：冻结与微调视觉编码器

[填写本日 Notebook 作业](../notebooks/week-06/day-02.ipynb)

[上一课：Day 1](day-01.md) · [本周首页](README.md) · [下一课：Day 3](day-03.md)

## 1. 今日目标与预计时间

预计 90 分钟。实现 frozen 与 fine-tune 两种训练模式，用参数与梯度审计证明模式真正生效，并制定低资源优先策略。

## 2. 这在 VLA 中的作用

冻结 encoder 能降低算力和过拟合风险；微调可能让视觉特征适应控制细节，但需要更多数据。二者差异常因错误的 `requires_grad`、优化器参数组或 train/eval 模式而被误判。

## 3. 核心概念、公式与数据流

若参数分为 encoder `θ_e` 和 action head `θ_h`：

```text
frozen:    ∂L/∂θ_e 不计算，optimizer 只含 θ_h
fine-tune: optimizer 含 θ_e 与 θ_h，通常 lr_e < lr_h
```

可设 `lr_e=0.1 lr_h`。注意 `requires_grad=False` 与 `model.eval()` 不等价：前者控制梯度，后者控制 dropout/normalization 行为。

## 4. 分步学习

1. 列出所有参数名、shape、参数量和 `requires_grad`。
2. frozen 模式中排除 encoder 参数，并明确是否保持 eval。
3. fine-tune 模式建两个 optimizer parameter groups。
4. 对同一 batch 各跑一步，比较 encoder/head 梯度范数与参数更新量。
5. 根据数据规模、算力和 domain gap 写选择结论；未下载预训练权重时把 tiny CNN “冻结”仅作为机制练习。

## 5. 必做作业

- 实现 `set_encoder_mode(frozen)` 或等价配置。
- 输出两种模式的 trainable 参数表、optimizer 组与学习率。
- 保存一步更新前后参数，验证 frozen encoder 不变、head 改变；fine-tune 两者按设定改变。

## 6. 输入、预期输出与验证

输入：Day 1 encoder、随机或真实 batch `[4,3,64,64]`、任意可反传标量 loss。

预期输出：frozen 时 encoder 梯度为 `None` 且参数最大差 0；head 参数差大于 0。fine-tune 时 encoder 至少一项梯度和更新非零。均为预期结果。

验证：比较 state_dict 克隆而非对象引用；确保 optimizer 中 parameter id 无重复；打印训练参数量和估算激活内存。

## 7. 固定提交格式

```markdown
# W6D2 提交
- 实际用时：
## 冻结/微调选择依据
## Trainable 参数与 optimizer 组
## 单步更新对照表
## 命令及完整日志
## 模式错误的负向测试
## 自测答案
```

## 8. 评分与验收

- 能区分 requires_grad 与 train/eval：20 分。
- 参数/优化器审计准确：25 分。
- frozen 更新验证：20 分。
- fine-tune 更新验证：20 分。
- 资源决策与自测：15 分。

总分 100，80 分通过；只声称“冻结”却没有参数差或梯度证据时不通过。

## 9. 常见错误与排查

| 错误 | 排查 |
|---|---|
| 冻结后仍在 optimizer | 检查 parameter id，不只看 requires_grad |
| state_dict 前后比较始终为零 | 更新前必须 clone/detach |
| eval 导致整个 policy 不训练 | 只对 encoder 切换模式，head 保持 train |
| 微调立刻发散 | 降 encoder lr、检查 normalization 与梯度范数 |

## 10. 时间不足时的最低完成线

在 tiny CNN 上完成 frozen/fine-tune 单步参数差验证，不要求长时间训练。

## 11. 可选提高

实现逐层解冻；比较 adapter/LoRA 风格的小参数更新；记录一步训练显存或 CPU 时间。

## 12. 完成后的自测题

1. `eval()` 为什么不能替代冻结？
2. frozen encoder 的输出是否仍可让 head 得到梯度？
3. 为什么 encoder 的学习率通常更小？
4. 无预训练 tiny CNN 冻结有什么局限？

[上一课：Day 1](day-01.md) · [本周首页](README.md) · [下一课：Day 3](day-03.md)
