# 第 7 周 Day 1：Tokenizer 与文本编码器

[填写本日 Notebook 作业](../notebooks/week-07/day-01.ipynb)

[上一课：第 6 周 Day 7](../week-06/day-07.md) · [本周首页](README.md) · [下一课：Day 2](day-02.md)

## 1. 今日目标与预计时间

预计 120 分钟。建立离线可复现的 tokenizer、词表和 tiny text encoder，正确处理 padding、未知词和 attention mask，并完成正反向测试。

## 2. 这在 VLA 中的作用

指令必须先变成固定语义的 token 序列。词表漂移、pad 被平均、训练/推理规范化不同都会让同一句话在 rollout 时得到不同表示。

## 3. 核心概念、公式与张量

```text
instruction strings [B]
→ token_ids [B,L] int64
→ valid_mask [B,L] bool
→ embeddings [B,L,D]
→ masked pooling / Transformer
→ z_text [B,D]
```

masked mean：

\[
z=\frac{\sum_{i=1}^{L}m_i e_i}{\max(1,\sum_i m_i)}.
\]

至少定义 `<pad>、<unk>、<bos>、<eos>`；词表只用训练文本建立。中文可选择字符级 tokenizer 以避免分词依赖；CPU 路径用 `Embedding + masked mean + MLP`。

## 4. 分步学习

1. 冻结文本规范化：空白、标点、大小写和数字处理。
2. 仅从训练指令建立词表，保存 token→id 和版本哈希。
3. 实现 batch encode、padding、truncation 和 mask。
4. 构建 tiny encoder，输入 token ids/mask，输出 token/pooled 特征。
5. 测同一句独立编码与在不同长度 batch 中编码是否一致。
6. 跑 forward/backward 并检查 embedding 的非 pad token 梯度。

## 5. 必做作业

- 提交词表、规范化规则、最大长度和特殊 token id。
- 写至少 6 个 tokenizer 测试：空串、未知字/词、长句截断、padding、一致性、批次重排。
- 输出 `token_ids [B,L]`、mask、`z_text [B,D]` shape 及梯度日志。

## 6. 输入、预期输出与验证

输入：至少 8 条结构化指令，如“把红色方块放进蓝色盒子”；另含一条训练词表外词和一条超长句。

预期输出：token ids 为 int64；pad mask 与长度一致；pooled feature 不受额外右侧 padding 影响；未知词映射 `<unk>` 而不崩溃。均为预期结果。

验证：词表保存再加载后编码完全相同；改变 pad embedding 不影响 masked pooled 输出；全 pad/空串按契约拒绝或映射 BOS/EOS，不产生 NaN。

## 7. 固定提交格式

```markdown
# W7D1 提交
- 实际用时：
## 规范化与词表策略
## 特殊 token/最大长度
## 8 条指令编码示例
## 六个测试与日志
## Encoder shape/梯度检查
## 自测答案
```

## 8. 评分与验收

- 词表来源、版本和规范化明确：20 分。
- tokenizer 六类测试：25 分。
- mask 与 pooling 数学正确：20 分。
- encoder forward/backward：20 分。
- 保存重载与自测：15 分。

总分 100，80 分通过；词表使用 test 指令构建、pad 参与无掩码平均或 checkpoint 不保存词表时不通过。

## 9. 常见错误与排查

| 错误 | 排查 |
|---|---|
| mask 语义反了 | 明确 True 表示 valid 还是 pad，并写手算小例 |
| batch 不同输出不同 | 检查 padding 是否进入 pooling/attention |
| 未知词导致越界 | 所有 OOV 映射固定 `<unk>` id |
| 文本 encoder 无梯度 | 查 detach、冻结和 optimizer 参数组 |

## 10. 时间不足时的最低完成线

用字符级词表 + Embedding masked mean，在 CPU 通过 padding、OOV、保存重载和非零梯度四项测试。

## 11. 可选提高

实现 tiny Transformer encoder；比较字符级与词级长度；对同义指令做表示相似度探索但不提前宣称语义泛化。

## 12. 完成后的自测题

1. 为什么词表只能由训练指令构建？
2. `<unk>` 与未见组合有什么本质区别？
3. padding mask 影响哪些计算？
4. 相同句子在不同 batch 中表示不同通常说明什么？

[上一课：第 6 周 Day 7](../week-06/day-07.md) · [本周首页](README.md) · [下一课：Day 2](day-02.md)
