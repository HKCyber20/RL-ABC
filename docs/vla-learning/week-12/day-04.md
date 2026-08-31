# 第 12 周 Day 4：最终训练或低资源确定性复跑

[填写本日 Notebook 作业](../notebooks/week-12/day-04.ipynb)

[← 上一日](day-03.md) · [本周首页](README.md) · [下一日 →](day-05.md)

## 1. 今日目标与预计时间

预计：**160–240 分钟**。

- 从冻结 config 训练/加载最终策略。
- 完成 smoke、短跑和正式运行三级证据。
- 记录 checkpoint 选择规则与资源成本。

## 2. 今日知识点及其在 VLA 中的作用

最终训练的价值在于可复跑和与 spec 一致；低资源路线也必须真实训练/加载小策略并闭环执行，不能只展示预期 loss。

## 3. 必要概念、公式、形状与数据流

- 训练证据链：config hash + data hash → run_id → step/epoch logs → checkpoint → selection rule；评估不能临时挑最好测试结果。
- BC 示例 loss 可为连续动作 MSE/L1 加 gripper CE；每项必须有 mask，shape 如 prediction/target `[B,D_a]` 或 `[B,H_a,D_a]`。
- 三级运行：单 batch overfit/smoke；短跑验证 loss/梯度/保存；正式预算运行。资源不足则缩小图像/模型/数据，不跳过闭环。

## 4. 分步骤学习内容

1. 运行数据与配置 preflight。
2. 单 batch 前向/反向并检查 finite、shape、mask。
3. 按预算运行训练，保存完整日志和最后/最佳 checkpoint 选择规则。
4. 从新进程加载 checkpoint，做至少一个闭环 rollout smoke。

## 5. 必做作业

- 提交训练命令、config/data hash、日志、checkpoint manifest 与资源用量。
- 展示至少一次失败运行及定位；没有失败则注入错误 stats key。
- 低资源路线提交实际小模型训练/加载证据；OpenVLA 仅需第 10 周 dry-run。

## 6. 作业输入、预期输出与验证方法

| 项目 | 内容 |
|---|---|
| 输入 | Day 1–3 冻结产物；CPU 可用小 CNN/MLP 或已实现语言条件 BC。 |
| 预期输出 | loss 有实际有限值且 checkpoint 可重载；闭环 smoke 实际执行；未运行的大模型路径仅列预期命令。 |
| 验证方法 | 新进程加载并对固定样本复现输出容差。；确认评估 checkpoint 按预注册规则选。；扫描 NaN/Inf、梯度和输出动作范围。 |

本周“预期”不能替代最终核心闭环实测；需要大模型/GPU 的 OpenVLA 部分可用明确标注的 CPU/mock dry-run。

## 7. 提交内容与固定格式

```markdown
# W12D04 提交

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

硬门槛：没有真实训练/加载或闭环 smoke，checkpoint 与 config/data hash 不匹配，或伪造 OpenVLA 运行，不得通过。

## 9. 常见错误与排查提示

- 只保存模型权重不保存预处理/stats。
- 用 test success 选择 checkpoint。
- 训练 loss 下降但 action 单位错。
- 先核对 spec/config/data/checkpoint/result hash、seed、stats key、动作单位/frame，再解释性能。
- 不得把手写 outcome、mock OpenVLA 输出或预期命令包装成真实闭环/模型结果。

## 10. 时间不足时的最低完成线

CPU 小策略真实训练、保存、重载和 1 个闭环 smoke；结果不要求高成功率。 最低完成线仍须满足当天硬门槛。

## 11. 可选提高任务

在授权资源下运行 OpenVLA processor/checkpoint 或 LoRA/OFT 最小步，并单独记录证据等级。

## 12. 完成后的自测题

1. 最终 checkpoint 如何选？
2. 单 batch overfit 能证明什么？
3. 低 loss 为什么仍需 rollout？

---

[← 上一日](day-03.md) · [本周首页](README.md) · [下一日 →](day-05.md)
