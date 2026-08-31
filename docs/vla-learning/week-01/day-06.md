# 第 1 周 Day 6：Toy Fusion Network 的前向与反向

[填写本日 Notebook 作业](../notebooks/week-01/day-06.ipynb)

[← 上日：Cross-Attention](day-05.md) · [周首页](README.md) · [次日：周测与项目 →](day-07.md)

## 1. 今日目标与预计时间

预计 2–3 小时。搭建一个小型 `image_tokens + text_tokens + robot_state -> action` 网络，完成形状断言、一次 forward、标量 loss 和 backward，确认跨模态路径可训练。

## 2. 今日知识点及其在 VLA 中的作用

今天把前五天连接成最小策略：视觉 token 提供 K/V，文本 token 提供 Q，cross-attention 输出经池化形成语义视觉特征；机器人状态经 MLP 编码；两者拼接后由动作头输出连续动作。真实 VLA 更复杂，但调试原则相同：先固定接口和形状，再验证梯度。

## 3. 必要概念、公式、形状与数据流

采用 `B=4,P=16,L=6,D=32,d_s=8,d_a=7,H=4`：

```text
image_tokens [4,16,32] -- K/V --+
text_tokens  [4, 6,32] -- Q ----+-> cross [4,6,32]
                                      mean over L -> [4,32]
state [4,8] -> state_mlp -> [4,16]
concat -> [4,48] -> action_head -> [4,7]
target_action [4,7] -> MSE -> scalar
```

\[
\mathcal L=\frac{1}{B d_a}\sum_{b,j}(\hat a_{b,j}-a_{b,j})^2
\]

`loss.backward()` 后至少动作头、cross-attention 投影和状态 MLP 的参与参数应具有非空、有限梯度。

## 4. 分步骤学习

1. 先定义构造参数并断言 `D % H == 0`。
2. 单独运行 cross-attention，检查 `[B,L,D]`。
3. 池化文本输出，编码状态并拼接。
4. 输出 `[B,7]`，与同形状 target 计算 MSE。
5. backward 后遍历参数，记录梯度是否存在及范数。
6. 故意将 state 改成 `[B,7]`，保存报错并解释接口契约如何发现问题，然后恢复。

## 5. 必做作业

用 PyTorch 实现 `ToyVLAPolicy`，模块至少含 cross-attention、状态 MLP 和动作头。设置随机种子，完成 forward/backward；加入不少于 5 个断言：三类输入、cross 输出、动作输出。提交模型代码、实际日志、参数量、一个错误注入案例。不要把随机 loss 数值写成课程预期，必须以实际运行为准。

## 6. 输入、预期输出与验证方法

- 输入：随机 image `[4,16,32]`、text `[4,6,32]`、state `[4,8]`、target `[4,7]`。
- 预期：prediction `[4,7]`、loss 为有限标量、至少三个模块的梯度范数有限且通常非零。
- 验证：固定 seed 后连续两次新建同模型应可复现；`torch.isfinite(loss)`；检查 `grad is not None`；执行一次 optimizer step 后至少一个参数改变。

低资源替代：CPU、小 batch `B=2`、`D=8,H=2`。没有 PyTorch 时提交完整伪代码和逐层形状，但本周只能记“理论通过”，之后需补运行证据。

## 7. 提交内容与固定格式

固定包含：`环境版本`、`模型结构`、`形状断言`、`forward/backward 日志`、`梯度表`、`错误注入与修复`、`低资源调整（如有）`、`投入分钟数`。

## 8. 评分与验收标准

100 分，80 分通过：结构与方向 20；形状断言 20；forward/loss 15；backward 梯度 25；错误注入 10；可复现记录 10。输出不是 `[B,7]`、状态未参与输出或无跨模态梯度任一出现即不通过。

## 9. 常见错误与排查

- `MultiheadAttention` 默认非 batch-first：显式设置或正确转置。
- 池化维错：文本序列维是 1，不应把 batch 求平均。
- 状态张量被 `.detach()`：检查计算图。
- 只检查动作头梯度：还需检查注意力和状态分支。
- optimizer step 前后对比的是同一引用：更新前 `clone()` 参数。

## 10. 时间不足时的最低完成线

运行到 `[B,7]` 前向、有限 loss 和一次 backward；错误注入及 optimizer step 可在 Day 7 补齐。

## 11. 可选提高任务

加入 2 个 learnable action query，令动作 query 读取拼接后的视觉与文本上下文，并比较输出语义与当前文本 Q 方案。

## 12. 完成后的自测题

1. 为什么状态分支不能只在 loss 之后拼接？
2. 哪个维度做 mean pooling？
3. 梯度非空是否保证模型学得好？
4. 随机输入实验能验证什么，不能验证什么？

[← 上日：Cross-Attention](day-05.md) · [周首页](README.md) · [次日：周测与项目 →](day-07.md)
