# 第 8 周 Day 4：端到端训练流水线与可复现配置

[填写本日 Notebook 作业](../notebooks/week-08/day-04.ipynb)

[上一课：Day 3](day-03.md) · [本周首页](README.md) · [下一课：Day 5](day-05.md)

## 1. 今日目标与预计时间

预计 2–3 小时。用单一配置驱动数据、模型、训练和 checkpoint；完成 dry run、小批过拟合、短训练、断点恢复和重载推理。

## 2. 这在 VLA 中的作用

最小 VLA 已跨越多种预处理与元数据。配置若无法追溯，任何成功率都无法复现；断点恢复若丢 optimizer/statistics，后续训练和 rollout 可能悄然改变。

## 3. 核心概念与流水线

```text
config + manifest + split
→ validate compatibility
→ train-only statistics
→ dataloaders
→ policy + optimizer
→ dry run / overfit
→ train & validate
→ checkpoint + metrics
→ reload predict
```

checkpoint 最少保存：模型、optimizer、scheduler（若有）、global_step、随机状态或 seed、config、split id、schema/tokenizer hash、state/action statistics。实验 id 可由关键配置哈希生成。

## 4. 分步学习

1. 整理一个配置文件，包含数据、输入 shape、encoder、fusion、chunk、loss、optimizer、seed、评估。
2. 启动时验证 schema/tokenizer/action stats 一致，不匹配即失败。
3. `--dry-run` 只取一批，跑 forward/loss/backward/optimizer 后退出。
4. `--overfit-small-batch` 复用固定小批，检查所有模态可学习。
5. 执行 CPU 短训练，定期保存 best/last 与 metrics JSONL/CSV。
6. 从 last 恢复若干步，比较 global_step 与 loss 连续性。
7. 新进程加载 best，对固定 batch 输出哈希/最大差。

## 5. 必做作业

- 提交完整配置和兼容性检查列表。
- 提供 dry run、小批过拟合、短训练、断点恢复、新进程推理五类实际日志。
- 对一处故意 metadata 不匹配做负向测试，确认训练在开始前失败。

## 6. 输入、预期输出与验证

输入：本周数据、连续动作策略；CPU 路径 `B≤8`、`64×64`、tiny encoder、数百样本和很短 epoch。

预期输出：dry run 退出码 0；小批 loss 显著下降；短训练写出指标/checkpoint；resume 从正确 step 继续；重载输出一致。具体数值需真实日志证明。

验证：检查 train/val episode 无交集；best 的选择只基于 val；测试数据不用于调参；恢复后 optimizer state 非空；固定 batch 预处理哈希一致。

## 7. 固定提交格式

```markdown
# W8D4 提交
- 实际用时与设备：
## 配置与资产哈希
## 兼容性检查清单
## Dry run/过拟合/短训练日志
## 断点恢复证据
## 新进程重载输出差
## Metadata 负向测试与自测
```

## 8. 评分与验收

- 配置与资产可追溯：20 分。
- dry run 和小批过拟合：20 分。
- 短训练/验证指标：15 分。
- resume 与重载一致：25 分。
- 负向测试、split 审计和自测：20 分。

总分 100，80 分通过；测试集参与选 best、checkpoint 缺 tokenizer/statistics 或 resume 实际从头开始却未说明时不通过。

## 9. 常见错误与排查

| 错误 | 排查 |
|---|---|
| dry run 通过但训练首 epoch 崩 | 检查 episode 尾部、空 mask 和变长 batch |
| resume loss 跳变 | 检查 optimizer/scheduler、随机状态和数据顺序 |
| best/last 混淆 | 文件名和 metadata 均记录 global_step/val 指标 |
| 重载动作不同 | 核对 eval、preprocess、词表、stats、chunk 配置 |

## 10. 时间不足时的最低完成线

CPU 完成 dry run、32 样本过拟合、保存/重载和一次 metadata mismatch；短训练可控制在几分钟并报告真实耗时。

## 11. 可选提高

加入确定性运行开关、自动实验清单、梯度/激活监控或配置 schema 验证。

## 12. 完成后的自测题

1. dry run 与小批过拟合分别验证什么？
2. best checkpoint 为什么不能用 test 选？
3. resume 最容易遗漏哪些状态？
4. 资产哈希能防止哪些静默错误？

[上一课：Day 3](day-03.md) · [本周首页](README.md) · [下一课：Day 5](day-05.md)
