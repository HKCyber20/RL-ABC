# 第 10 周 Day 5：动作反归一化与安全边界

[填写本日 Notebook 作业](../notebooks/week-10/day-05.ipynb)

[← 上一日](day-04.md) · [本周首页](README.md) · [下一日 →](day-06.md)

## 1. 今日目标与预计时间

预计：**120–160 分钟**。

- 实现 normalized action 到物理动作的可逆映射。
- 绑定 dataset key、统计版本与动作 mask。
- 在下发 controller 前做范围、变化率和有限值检查。

## 2. 今日知识点及其在 VLA 中的作用

视觉推理正确也可能因统计 key 错而产生危险动作；反归一化是 OpenVLA 接入中必须独立验收的安全关键层。

## 3. 必要概念、公式、形状与数据流

- percentile 映射：a=(z+1)/2*(p99-p01)+p01；若训练使用 mask，仅对有效维变换，离散 gripper 按契约解码。
- stats 至少包含 key/version/action_names/p01/p99/mask/unit/frame/control_hz；key 不存在或维度不匹配应 fail closed。
- 部署前检查 isfinite、逐维范围、单步 delta、gripper 合法值；clip 只能是显式安全策略，不能掩盖统计错误。

## 4. 分步骤学习内容

1. 创建两个不同范围的 mock dataset stats。
2. 实现 normalize_action 和 unnormalize_action。
3. 测试 round-trip、未知 key、常量维、mask、越界 normalized action。
4. 实现 controller guard，并区分 reject 与 explicit clip。

## 5. 必做作业

- 提交 action_stats.json、unnormalize_action.py、不少于 8 个测试和日志。
- 证明同一个 z 在两个 key 下得到不同物理动作。
- 写一次错误 key 的潜在后果分析。

## 6. 作业输入、预期输出与验证方法

| 项目 | 内容 |
|---|---|
| 输入 | z=[0,-1,1,0.5,0,0,1]；两个 key 的平移范围分别 [-0.02,0.02] m 与 [-0.05,0.05] m。 |
| 预期输出 | round-trip 最大误差 <1e-6；未知 key/维度错/NaN 均拒绝；输出逐维带名称、单位与 frame。 |
| 验证方法 | 断言 round-trip。；交换 key 并比较物理输出。；注入 NaN 与极端值，确认 controller guard 阻断。 |

真实模型路径如需联网下载、安装或 GPU，必须先说明目的、磁盘/显存/时间成本并获得授权；默认完成低资源路径。

## 7. 提交内容与固定格式

```markdown
# W10D05 提交

- 实际用时：记录分钟数
- 证据级别：静态检查 / mock dry-run / 真实 processor / 真实 checkpoint（只选实际完成项）
- 环境与 revision：记录版本；未知项明确写“未知”

## 概念与接口
记录公式、shape、dataset key 和模块边界。

## 实现与配置
列出文件路径、配置摘要和设计理由。

## 验证证据
粘贴命令、退出码、关键输出；没有运行的输出标为“预期”。

## 失败注入与安全处理
说明错误在哪一层被阻断，附修复证据。

## 自测答案
逐题作答。
```

## 8. 评分与验收标准

| 评分项 | 分值 | 得分条件 |
|---|---:|---|
| 架构/公式/shape | 20 | 数据流准确，稳定概念与版本项分开 |
| 接口或配置产物 | 25 | 输入输出、异常、revision 与 key 完整 |
| 验证证据 | 25 | 命令/断言/输出可复核，mock 与 real 明确区分 |
| 动作与安全分析 | 20 | 反归一化、单位、frame、范围或 controller 风险正确 |
| 复盘与自测 | 10 | 能定位失败并回答自测 |
| **总分** | **100** | **80 分通过** |

硬门槛：未知 key 使用默认统计、缺单位/frame，或动作未经 guard 直接下发，不得通过。

## 9. 常见错误与排查提示

- 把 p01/p99 顺序写反。
- gripper 连续维被不恰当地按平移范围缩放。
- clip 后仍报告原预测无异常。
- 先打印输入 mode/shape、processor keys、model revision、action stats key 与 action shape，再排查数值。
- 不得声称执行过未实际运行的安装、推理或训练。

## 10. 时间不足时的最低完成线

一个 stats key、7 维 round-trip、未知 key 和 NaN 两个拒绝测试。 最低线也要提交实际证据或逐项“预期”标记。

## 11. 可选提高任务

给 stats 文件加 hash，并在 trace 中验证 hash。

## 12. 完成后的自测题

1. 为什么同一 z 不能跨机器人直接执行？
2. clip 与 reject 各适合什么情况？
3. 常量维如何处理？

---

[← 上一日](day-04.md) · [本周首页](README.md) · [下一日 →](day-06.md)
