# 第 12 周 Day 5：最终闭环评估与结果冻结

[填写本日 Notebook 作业](../notebooks/week-12/day-05.ipynb)

[← 上一日](day-04.md) · [本周首页](README.md) · [下一日 →](day-06.md)

## 1. 今日目标与预计时间

预计：**180–260 分钟**。

- 执行 ID 与至少两类 OOD 多 seed 评估。
- 从 episode manifest 生成成功率、CI 和失败分布。
- 冻结结果并进行独立重算。

## 2. 今日知识点及其在 VLA 中的作用

这是最终 Goal 的核心证据：策略必须在环境反馈下连续决策，结果既包含总体表现，也揭示泛化边界。

## 3. 必要概念、公式、形状与数据流

- 闭环循环：`reset(seed) → observe(o_t,s_t) → policy(image,instruction,state) → guard(a_t) → env.step → success/timeout`。
- 每格报告 k/n、Wilson CI、per-seed、平均步数/安全中止；总体值不能替代切片。
- 结果冻结保存 eval config、checkpoint hash、episode manifest、metrics 与生成脚本 hash。

## 4. 分步骤学习内容

1. 先跑每格 1 episode 的 eval smoke。
2. 按冻结预算执行 ID、未见位置和未见组合/背景至少两 OOD。
3. 运行失败 taxonomy 标注并抽查 trace。
4. 从原始 manifest 重算表，冻结 results id。

## 5. 必做作业

- 提交 FINAL_RESULTS.md、episode manifest、metrics.json/csv 与评估日志。
- 至少 3 seed；资源不足无法达到时如实报告，并增加每 seed episode 但结论降级。
- 选 3 个成功和 5 个失败案例，附关键 step 证据。

## 6. 作业输入、预期输出与验证方法

| 项目 | 内容 |
|---|---|
| 输入 | Day 4 checkpoint 与 Week 11 matrix；必须来自实际闭环仿真，不能用手写 mock outcome。 |
| 预期输出 | ID+两类 OOD 的切片表、CI、per-seed 和失败分布；每个数字可回溯 episode。 |
| 验证方法 | 随机重算两个 cell。；独立检查 manifest 唯一 id 和分母。；重播/重查至少一个成功和一个失败 trace。 |

本周“预期”不能替代最终核心闭环实测；需要大模型/GPU 的 OpenVLA 部分可用明确标注的 CPU/mock dry-run。

## 7. 提交内容与固定格式

```markdown
# W12D05 提交

- 实际用时：记录分钟数
- run/result id：记录可追踪标识
- 证据等级：真实轻量/完整仿真、真实 OpenVLA、OpenVLA mock dry-run（逐项说明）

## 冻结合同与设计
记录 spec/config/data/checkpoint hash 及关键 shape、单位和 frame。

## 实际产物
列出代码、数据、日志、报告和 demo 的绝对或仓库相对路径。

## 复现与验证证据
粘贴命令、退出码、关键输出、重算/重播证据；未执行项明确写“预期”。

## 失败、限制与补救
区分观察、推断和假设，给最小补救验收标准。

## 自测/答辩答案
逐题作答。
```

## 8. 评分与验收标准

| 评分项 | 分值 | 得分条件 |
|---|---:|---|
| spec/概念/shape | 20 | 输入、动作、闭环和版本合同准确 |
| 实际项目产物 | 25 | 文件完整，来源与 hash 可追踪 |
| 复现与验证 | 25 | 实际命令、退出码、重跑/重算证据充分 |
| 泛化/失败/OpenVLA | 20 | 证据边界清楚，能诊断并解释接入 |
| 复盘与答辩 | 10 | 限制诚实，回答自测并给补救 |
| **总分** | **100** | **80 分通过** |

硬门槛：核心结果不是实际闭环、没有 episode 级证据、缺 OOD 或只报最好 seed，均不得通过。

## 9. 常见错误与排查提示

- 评估时启用训练增强。
- timeout 被排除分母。
- 复跑时环境初态列表改变。
- 先核对 spec/config/data/checkpoint/result hash、seed、stats key、动作单位/frame，再解释性能。
- 不得把手写 outcome、mock OpenVLA 输出或预期命令包装成真实闭环/模型结果。

## 10. 时间不足时的最低完成线

轻量仿真中实际运行 3 seed，ID+两类 OOD 每类至少 9 episode，并完成失败分类。 最低完成线仍须满足当天硬门槛。

## 11. 可选提高任务

加入 paired checkpoint 比较或 action smoothness/碰撞率。

## 12. 完成后的自测题

1. 最终主结论对应哪个表？
2. 最差 OOD 是什么？
3. 结果冻结后发现 bug 怎么处理？

---

[← 上一日](day-04.md) · [本周首页](README.md) · [下一日 →](day-06.md)
