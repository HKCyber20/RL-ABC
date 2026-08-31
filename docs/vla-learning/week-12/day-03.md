# 第 12 周 Day 3：数据卡、manifest 与质量签名

[填写本日 Notebook 作业](../notebooks/week-12/day-03.ipynb)

[← 上一日](day-02.md) · [本周首页](README.md) · [下一日 →](day-04.md)

## 1. 今日目标与预计时间

预计：**130–180 分钟**。

- 完成最终数据卡与样本级 manifest。
- 证明图像、语言、状态和动作时间对齐。
- 冻结 split、归一化统计和质量报告。

## 2. 今日知识点及其在 VLA 中的作用

数据卡把“训练用了什么”变成可审计事实，是跨机器复现与解释泛化边界的基础。

## 3. 必要概念、公式、形状与数据流

- Data Card 包含来源/生成、许可、任务、本体/环境、episode/step 数、字段 shape/dtype、动作、频率、split、统计、质量过滤、偏差、限制。
- manifest 每个 episode 至少含 id、split、seed、task、length、success/demo quality、source、hash；统计只从 train split 计算。
- 质量门：边界标志、时间戳容差、shape、finite、动作范围、语言非空、重复/泄漏、最后 step 过滤。

## 4. 分步骤学习内容

1. 生成数据清单并核对 episode/step 口径。
2. 运行 inspector，输出错误码计数与过滤前后数量。
3. 计算 train-only stats 并保存 key/version/hash。
4. 做 train/test episode id、场景种子和组合泄漏检查。

## 5. 必做作业

- 提交 DATA_CARD.md、dataset_manifest、quality_report 与 stats。
- 人工抽查至少 3 个 episode 的 `o_t→a_t→o_{t+1}`。
- 故意交换一条 split 或动作帧，证明检查器拒绝后再修复。

## 6. 作业输入、预期输出与验证方法

| 项目 | 内容 |
|---|---|
| 输入 | 冻结 spec、最终演示数据；低资源环境也必须是真实生成的轻量仿真轨迹，不用手写结果代替。 |
| 预期输出 | 所有计数一致；train/test 无声明之外泄漏；stats 只来自 train；错误注入有检测与修复证据。 |
| 验证方法 | 从 manifest 重算 episode/step 数。；重算一个动作维的统计。；检查 hash 在训练 config 中被引用。 |

本周“预期”不能替代最终核心闭环实测；需要大模型/GPU 的 OpenVLA 部分可用明确标注的 CPU/mock dry-run。

## 7. 提交内容与固定格式

```markdown
# W12D03 提交

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

硬门槛：统计使用 test 数据、时间对齐无证据、或数据来源/生成过程不明，不得通过。

## 9. 常见错误与排查提示

- 把 step 数写成 episode 数。
- 过滤后未更新 manifest/hash。
- 重复 episode 跨 split。
- 先核对 spec/config/data/checkpoint/result hash、seed、stats key、动作单位/frame，再解释性能。
- 不得把手写 outcome、mock OpenVLA 输出或预期命令包装成真实闭环/模型结果。

## 10. 时间不足时的最低完成线

10 个真实轻量仿真 episode、数据卡、manifest、5 条质量检查与 train-only stats。 最低完成线仍须满足当天硬门槛。

## 11. 可选提高任务

加入图像缩略图索引和覆盖率可视化。

## 12. 完成后的自测题

1. 为什么 stats 只能用 train？
2. 数据卡与 README 有何区别？
3. 怎样发现重复轨迹泄漏？

---

[← 上一日](day-02.md) · [本周首页](README.md) · [下一日 →](day-04.md)
