# 第 10 周 Day 7：LoRA、OFT、全参微调取舍与周项目

[填写本日 Notebook 作业](../notebooks/week-10/day-07.ipynb)

[← 上一日](day-06.md) · [本周首页](README.md) · [下一日 →](../week-11/day-01.md)

## 1. 今日目标与预计时间

预计：**160–220 分钟**。

- 基于目标、资源和部署频率选择适配方案。
- 完成不加载 checkpoint 的 config/dataloader/forward-contract dry-run。
- 验收整条 OpenVLA 接入模拟管线。

## 2. 今日知识点及其在 VLA 中的作用

最终项目需要的是可复现决策与证据，而不是盲目追求最大训练；本日把适配方式、资源限制和接口风险收敛成可执行方案。

## 3. 必要概念、公式、形状与数据流

- LoRA 只训练低秩适配参数，节省可训练参数/优化器成本，但仍需基础模型；全参微调资源与遗忘风险更高。
- OFT 是面向 VLA 的优化微调路线，强调连续动作、action chunk/并行解码等改进；具体支持与参数以所用官方仓库版本为准，不能与原始离散 action LoRA 配置直接混用。
- dry-run 分级：配置解析 → dataset/adapter → 单 batch shape → mock forward/loss contract → 可选真实单步；每一级单独声明证明范围。

## 4. 分步骤学习内容

1. 完成 LoRA/OFT/全参四维决策表：目标、训练资源、推理接口、风险。
2. 冻结一个适配配置，含 model revision、dataset、stats key、batch、seed、输出目录。
3. 用 mock model 跑一批数据，验证 labels/mask/shape 与 loss 为有限标量。
4. 复跑完整 pipeline 并记录 go/no-go。

## 5. 必做作业

- 提交 adaptation_decision.md、dry-run 配置、命令/预期命令与 w10_week_project.md。
- 周项目覆盖两条合法样本和至少 5 类失败输入。
- 若资源允许且获得授权，可另附真实 checkpoint 证据；否则 mock 通过不降分，但必须准确界定。

## 6. 作业输入、预期输出与验证方法

| 项目 | 内容 |
|---|---|
| 输入 | Day 1–6 全部产物；默认 CPU、无网络、无 checkpoint。 |
| 预期输出 | trace 为 adapter→processor→mock model→unnormalize→guard；决策表能解释为何当前选择/暂缓某方案；所有结果来源标明 mock/real。 |
| 验证方法 | 删除 model revision 或 stats key，配置校验失败。；复跑固定 seed 的 mock trace 一致。；逐一核对 5 类失败均在正确层阻断。 |

真实模型路径如需联网下载、安装或 GPU，必须先说明目的、磁盘/显存/时间成本并获得授权；默认完成低资源路径。

## 7. 提交内容与固定格式

```markdown
# W10D07 提交

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

硬门槛：把 config parse 称作训练成功、把 mock forward 称作 OpenVLA 推理，或没有动作安全检查，不得通过。

## 9. 常见错误与排查提示

- LoRA 训练参数少被误解为无需大模型显存。
- 旧版命令参数直接套用到新配方。
- dry-run 没有 batch 与 label mask 检查。
- 先打印输入 mode/shape、processor keys、model revision、action stats key 与 action shape，再排查数值。
- 不得声称执行过未实际运行的安装、推理或训练。

## 10. 时间不足时的最低完成线

完成决策表、静态配置校验、一个 mock batch 与三个失败输入。 最低线也要提交实际证据或逐项“预期”标记。

## 11. 可选提高任务

获得授权后运行真实 processor（不加载权重）并比较 mock/real 输出键。

## 12. 完成后的自测题

1. 当前资源下为何选择该适配方案？
2. dry-run各层分别证明什么？
3. 真实推理前还缺哪三项证据？

---

[← 上一日](day-06.md) · [本周首页](README.md) · [下一日 →](../week-11/day-01.md)
