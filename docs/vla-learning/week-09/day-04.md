# 第 9 周 Day 4：RLDS：episode/step schema 与时间语义

[填写本日 Notebook 作业](../notebooks/week-09/day-04.ipynb)

[← 上一日](day-03.md) · [本周首页](README.md) · [下一日 →](day-05.md)

## 1. 今日目标与预计时间

预计：**100–130 分钟**。

- 准确解释 RLDS 的 episode 与 step 层级。
- 正确处理 `is_first`、`is_last`、`is_terminal` 和末步无效动作。
- 建立 `o_t,a_t,o_{t+1}` 时间对齐检查。

## 2. 今日知识点及其在 VLA 中的作用

RLDS 是 OXE/OpenVLA 数据接入的重要组织方式；边界语义错误会制造系统性的错误监督，比单个坏像素更危险。

## 3. 必要概念、公式与数据流

- RLDS 顶层是 episode dataset；每个 episode 含 step dataset 和可选 metadata。
- `is_first` 标记初始 step；`is_last` 标记 episode 最后 observation，该步之后的 action/reward/discount 通常无意义；`is_terminal` 表示环境终止，若 `is_last=true,is_terminal=false` 通常是截断。
- 监督样本应明确采用 `(observation_t, action_t)`，且最后 step 是否过滤必须符合数据生成契约。

## 4. 分步骤学习内容

1. 写出一个 4-step episode 的时序表。
2. 为成功终止与时间截断分别填写三个边界标志。
3. 创建 `mock_rlds_episode.json`，包含 observation、action、language 和 metadata。
4. 写 5 条断言检查首尾标志、终止关系、step 数与末步动作使用规则。

## 5. 必做作业

- 提交 2 个 episode：一个 terminal，一个 truncated。
- 实现 `validate_episode(ep)`，返回结构化错误列表。
- 故意制造 `is_last=false` 的坏样例并记录检查器输出。

## 6. 作业输入、预期输出与验证方法

| 项目 | 内容 |
|---|---|
| 输入 | 每步 observation 含 `image_shape:[3,64,64]`、`state:[8]`；有效 action 为 `[7]`；最后 step 的 action 设为 `null`。 |
| 预期输出 | 合法 episode 返回空错误列表；坏样例至少报告边界标志和末步语义错误；时序表明确动作作用于当前 observation 后产生下一 observation。 |
| 验证方法 | 断言恰有一个 `is_first` 且位于索引 0。；断言恰有一个 `is_last` 且位于末索引。；断言 `is_terminal → is_last`；截断样例允许末步 `is_terminal=false`。 |

预期输出只有在实际运行并保留证据后才能改写为“实测通过”。

## 7. 提交内容与固定格式

```markdown
# W09D04 提交

- 实际用时：写明分钟数
- 使用环境：写明 Python/库版本；未运行写“仅静态分析”

## 概念与推导
写出关键定义、公式推导和张量形状。

## 作业实现
列出文件路径、核心代码说明与设计选择。

## 验证证据
粘贴命令、退出码、关键输出；未执行的结果逐项标为“预期”。

## 错误注入与修正
记录至少一个失败、定位证据与修复后结果。

## 自测答案
逐题回答本页自测题。
```

## 8. 评分与验收标准

| 评分项 | 分值 | 得分条件 |
|---|---:|---|
| 概念、公式与形状 | 20 | 定义准确，变量、单位和 shape 可追踪 |
| 实现或结构化产物 | 25 | 文件完整，输入输出契约明确 |
| 验证证据 | 25 | 有命令/手算、断言、实际输出或诚实的“预期”标记 |
| VLA 场景分析 | 20 | 能联系闭环、数据或动作风险，而非复述定义 |
| 复盘与自测 | 10 | 解释失败原因并正确回答自测 |
| **总分** | **100** | **80 分通过** |

硬门槛：把 `is_last` 与成功等同，或将最后 observation 的 null action 作为训练标签，不得通过。

## 9. 常见错误与排查提示

- 混淆记录顺序和加载后的 step 对齐。
- 把 truncated 误判为失败或 terminal。
- 所有 episode 共用同一个可变 metadata 对象。
- 若输出与预期不符，先打印 episode/step 索引、shape、dtype、单位和 dataset key，再检查模型。
- 不得把论文中的报告结果或本页“预期输出”写成本地实测。

## 10. 时间不足时的最低完成线

完成两个 4-step JSON、边界表和至少 5 条检查。 最低完成线仍须提交证据，且只能获得“最低线通过”，后续需补齐完整产物。

## 11. 可选提高任务

增加 episode_id 唯一性与 invalid episode 过滤。

## 12. 完成后的自测题

1. `is_last=true,is_terminal=false` 说明什么？
2. 为什么最后 step 的 observation 仍可能有用？
3. 如何检测动作错位一帧？

---

[← 上一日](day-03.md) · [本周首页](README.md) · [下一日 →](day-05.md)
