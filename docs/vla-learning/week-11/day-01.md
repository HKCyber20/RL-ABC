# 第 11 周 Day 1：冻结评估矩阵与成功判据

[填写本日 Notebook 作业](../notebooks/week-11/day-01.ipynb)

[← 上一日](../week-10/day-07.md) · [本周首页](README.md) · [下一日 →](day-02.md)

## 1. 今日目标与预计时间

预计：**100–140 分钟**。

- 定义 ID/OOD 因素、水平、split 与 episode 预算。
- 冻结成功、失败、超时和无效运行的计数规则。
- 建立从 aggregate 指标回溯 episode 的 manifest。

## 2. 今日知识点及其在 VLA 中的作用

评估矩阵先于运行冻结，才能避免看到结果后挑条件；它也是比较 checkpoint、消融和泛化结论的共同坐标系。

## 3. 必要概念、公式、形状与数据流

- 评估单元是 episode；记录 `episode_id, checkpoint, config_hash, seed, condition, task, success, failure_code, steps`。
- 成功率 `p̂=k/n`，其中 n 必须说明是否包含 simulator crash、超时和安全中止；无效运行应有预先规则。
- 因素示例：position、object-color/shape、instruction composition、background；ID/OOD 必须由训练 manifest 判定而非主观印象。

## 4. 分步骤学习内容

1. 写唯一一句可程序化的任务成功判据。
2. 列出 4 个因素及各水平，标注训练覆盖。
3. 设计最小平衡矩阵，固定 seed 与每格 episode 数。
4. 定义异常、重试、最大步数和 checkpoint 选择规则。

## 5. 必做作业

- 提交 eval_matrix.md/yaml 与 episode_manifest schema。
- 构造 8 条样例记录，包含成功、任务失败、超时和无效运行。
- 写聚合伪代码，证明分母规则可执行。

## 6. 作业输入、预期输出与验证方法

| 项目 | 内容 |
|---|---|
| 输入 | 训练条件清单、策略 checkpoint/config；无 rollout 时可用明确标记 mock 记录。 |
| 预期输出 | 矩阵每格有 condition id、split、seed、预算和成功判据；任一汇总数字能回到 episode。 |
| 验证方法 | 检查 OOD 标签与训练 manifest。；对 8 条样例手算至少两个切片成功率。；修改分母规则并展示结果为何变化，但不在正式评估后更改冻结规则。 |

若未运行真实仿真，所有 mock 数字必须显式标为“仅验证分析管线”。

## 7. 提交内容与固定格式

```markdown
# W11D01 提交

- 实际用时：记录分钟数
- 数据证据：实际 rollout / 历史日志 / mock 分析管线（如实选择）
- checkpoint、config hash、seed：逐项记录，未知则说明

## 协议与公式
写成功判据、split、分母、公式和假设。

## 产物与数据
列出脚本、manifest、报告或图表路径。

## 验证证据
粘贴命令、退出码、关键输出和重算结果；未执行写“预期”。

## 失败与限制
记录错误注入、混杂变量、限制和补救。

## 自测答案
逐题作答。
```

## 8. 评分与验收标准

| 评分项 | 分值 | 得分条件 |
|---|---:|---|
| 协议、公式与统计 | 20 | split、分母、假设和公式准确 |
| 代码/数据/报告产物 | 25 | schema 完整，结果可回溯 episode |
| 验证与重算证据 | 25 | 有命令、输出、边界测试或人工复核 |
| 泛化与失败分析 | 20 | 控制混杂，区分观察/推断/假设 |
| 复盘与自测 | 10 | 诚实说明限制并答对自测 |
| **总分** | **100** | **80 分通过** |

硬门槛：未冻结成功/分母规则、没有 episode id，或结果后挑选 checkpoint，不得通过。

## 9. 常见错误与排查提示

- 把物体放到盒子边缘但未满足稳定时间也算成功。
- sim crash 被静默删除。
- ID 与 OOD episode 数严重不平衡却只报总体值。
- 先检查 manifest 行数、唯一 episode id、seed/condition 分布、缺失值和分母，再解释模型。
- 不得将 mock 记录、预期 CI 或未运行的 rollout 写成真实性能。

## 10. 时间不足时的最低完成线

完成 2×2 矩阵、8 条样例与明确分母。 最低线仍需固定协议并提交可复核证据。

## 11. 可选提高任务

加入预注册表，记录主要/次要指标和停止规则。

## 12. 完成后的自测题

1. 一个 eval cell 的最小描述包含什么？
2. 无效运行怎样处理才不偏？
3. 为什么先冻结 checkpoint？

---

[← 上一日](../week-10/day-07.md) · [本周首页](README.md) · [下一日 →](day-02.md)
