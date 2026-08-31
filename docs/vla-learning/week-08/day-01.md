# 第 8 周 Day 1：总体架构与接口冻结

[填写本日 Notebook 作业](../notebooks/week-08/day-01.ipynb)

[上一课：第 7 周 Day 7](../week-07/day-07.md) · [本周首页](README.md) · [下一课：Day 2](day-02.md)

## 1. 今日目标与预计时间

预计 120 分钟。画出并冻结最小 VLA 的模块、数据类型和 shape 接口，让一个 synthetic batch 从原始输入通过所有编码器、融合层和动作头。

## 2. 这在 VLA 中的作用

端到端集成最常见的错误并非模型公式，而是训练/推理预处理不同、mask 方向相反、统计缺失或动作定义漂移。接口冻结把这些隐性假设变为自动检查的合同。

## 3. 核心概念、公式与数据流

建议模型边界：

```text
rgb         [B,3,H,W] float32
token_ids   [B,L]     int64
text_mask   [B,L]     bool
state       [B,d_s]   float32
             ↓ encoders/fusion
action_pred [B,H_a,d_a] float32
```

单步动作是 `H_a=1` 的特例。定义 `policy(batch) → normalized_action_chunk`；环境 adapter 负责反归一化、坐标映射、边界检查和 step。checkpoint 必须绑定 `schema_version/tokenizer_hash/action_stats/config`。

## 4. 分步学习

1. 从第 5–7 周复制“契约结论”，不要重写另一套预处理。
2. 画环境、dataset、collate、policy、action adapter、success checker 的方向图。
3. 为每个模块写输入输出 key、shape、dtype、单位、设备和所有者。
4. 定义 `VLAPolicy.forward` 与 `predict_action`；后者必须切 eval/no_grad，但不做环境私有变换。
5. 用 `B=2` synthetic batch 完成 forward，加入至少 10 个断言。
6. 故意破坏 mask、shape、schema_version，确认错误在边界被捕获。

## 5. 必做作业

- 提交总体架构图和模块契约表。
- 实现端到端 smoke test，打印各中间张量 shape/有限值/设备。
- 实现三类负向测试：文本 mask 错、state 维错、checkpoint/schema 不匹配。

## 6. 输入、预期输出与验证

输入：两张任务相关 synthetic RGB、两条 tokenized 指令、两个 state；动作维使用已冻结 schema，chunk 暂设 `H_a=1`。

预期输出：动作 `[2,1,d_a]` 有限；所有模块 batch 一致；三种坏输入给出可定位异常。均为预期结果。

验证：模型 `eval()` 两次输出一致；batch 重排后输出同步重排；checkpoint metadata 与运行时 config 比较；动作输出不能在模型内默默 clip。

## 7. 固定提交格式

```markdown
# W8D1 提交
- 实际用时：
## 端到端架构图
## 模块契约表
## 10 项 shape/dtype/metadata 断言
## Smoke test 命令与日志
## 三个负向测试
## 自测答案
```

## 8. 评分与验收

- 架构覆盖数据到环境闭环：20 分。
- 模块契约六要素明确：20 分。
- 十项断言和 smoke test：25 分。
- 三种负向测试：20 分。
- 复现元数据与自测：15 分。

总分 100，80 分通过；训练与推理预处理责任不明、动作 shape/坐标不明或 schema 不匹配仍静默运行时不通过。

## 9. 常见错误与排查

| 错误 | 排查 |
|---|---|
| `[B,d_a]` 与 `[B,1,d_a]` 混用 | 契约固定保留 chunk 维 |
| mask 在不同库语义相反 | adapter 边界显式转换并做已知小例 |
| model 内外重复归一化 | 为 raw/normalized 命名并只指定一个所有者 |
| checkpoint 加载“成功”但词表不同 | 比较 tokenizer hash 与 schema version |

## 10. 时间不足时的最低完成线

CPU synthetic batch 跑通 image/text/state → `[2,1,d_a]`，完成 10 个断言和一个 schema mismatch 负向测试。

## 11. 可选提高

使用 dataclass/TypedDict 表达 batch；导出 shape trace；统计每个模块参数量和延迟。

## 12. 完成后的自测题

1. 哪些变换属于 dataset，哪些属于 environment adapter？
2. 为什么 action 保留 chunk 维更稳妥？
3. tokenizer hash 为何要进入 checkpoint？
4. 为什么模型内部不应默默 clip 物理动作？

[上一课：第 7 周 Day 7](../week-07/day-07.md) · [本周首页](README.md) · [下一课：Day 2](day-02.md)
