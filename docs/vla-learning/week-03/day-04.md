# 第 3 周 Day 4：训练/验证/测试划分与泄漏

[填写本日 Notebook 作业](../notebooks/week-03/day-04.ipynb)

[← 上日：损失与归一化](day-03.md) · [周首页](README.md) · [次日：协变量偏移 →](day-05.md)

## 1. 今日目标与预计时间

预计 90–120 分钟。按 episode 或场景划分数据，识别随机 transition 划分造成的近重复泄漏，并生成可审计的 split manifest。

## 2. 今日知识点及其在 VLA 中的作用

同一 episode 相邻帧高度相似。随机打散 transition 会让训练与验证共享几乎相同状态，验证 loss 过于乐观。VLA 的泛化还可能要求按物体布局、颜色组合或场景 seed 划分；划分单位必须与研究问题一致。

## 3. 必要概念、公式、形状与数据流

目标是三组 episode id 不相交：

\[
E_{train}\cap E_{val}=E_{train}\cap E_{test}=E_{val}\cap E_{test}=\varnothing
\]

manifest 至少记录：split seed、划分单位、episode id、场景 id、任务标签、transition 数、归一化统计来源。训练/验证用于模型选择，测试只用于最终评估。

按任务层级可有：

```text
IID: episode 随机但不交叉
unseen position: 按位置区间分组
unseen combination: 按颜色-目标组合留出
```

## 4. 分步骤学习

1. 构造 12 个 episode，每个带 scene_id 和长度。
2. 对比 transition-level 与 episode-level 划分。
3. 实现集合交集断言和样本数统计。
4. 设计一个 unseen-region 测试 split。
5. 保存 manifest 并从它重建同一划分。

## 5. 必做作业

创建 12 个 episode 的元数据，按 8/2/2 episode 划分；生成 `split_manifest`（可为 Markdown/JSON 展示，但本课只要求课程外作业产物）。提交三组 id、transition 数和交集检查。再模拟错误的 transition 随机划分，找出至少一个跨 split 的 episode id，说明验证偏乐观的机制。最后设计一个按目标 x 坐标留出区间的 OOD test。

## 6. 输入、预期输出与验证方法

- 输入：12 个唯一 episode id、每个长度与目标位置、固定 seed。
- 预期：8/2/2 个 episode；三组交集为空；总 transition 数守恒；相同 seed 重建一致。
- 验证：`union == all_ids` 且无重复；错误划分应能被 episode 泄漏审计捕获；train-only 统计 id 与 manifest 对齐。

低资源方案：纸面列出 12 行元数据并手工做集合检查。

## 7. 提交内容与固定格式

固定包含：`研究问题/划分单位`、`manifest`、`集合断言`、`样本数守恒`、`错误划分泄漏证据`、`OOD设计`、`投入分钟数`。

## 8. 评分与验收标准

100 分，80 分通过：划分理由 20；manifest 20；无泄漏验证 25；错误案例 20；OOD 设计 15。任何 episode 跨 split、归一化含测试数据、test 参与调参均为关键失败。

## 9. 常见错误与排查

- 比例按 transition 而非 episode 误读：同时报告两者。
- 场景相同但 episode 不同仍泄漏：若评估 unseen scene，应按 scene_id 分组。
- 保存 seed 不保存具体 id：库版本变化仍可能改变划分，manifest 需保存 id。
- 反复查看 test 选择超参：应设 validation，test 最终一次使用。

## 10. 时间不足时的最低完成线

完成 8/2/2 episode 列表、三组交集断言、错误 transition 划分的一个泄漏证据。

## 11. 可选提高任务

实现 group-stratified split，在不跨 episode/scene 的前提下平衡成功标签或任务类别，并说明无法完美平衡时的取舍。

## 12. 完成后的自测题

1. 为什么相邻帧随机划分尤其危险？
2. 按 episode 无泄漏是否自动代表按场景无泄漏？
3. validation 与 test 的职责有何不同？
4. split manifest 为什么比只保存 seed 更可靠？

[← 上日：损失与归一化](day-03.md) · [周首页](README.md) · [次日：协变量偏移 →](day-05.md)
