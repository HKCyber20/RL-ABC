# 第 10 周 Day 6：自定义 dataset adapter 与样例 batch

[填写本日 Notebook 作业](../notebooks/week-10/day-06.ipynb)

[← 上一日](day-05.md) · [本周首页](README.md) · [下一日 →](day-07.md)

## 1. 今日目标与预计时间

预计：**130–180 分钟**。

- 把自定义 episode 映射为 OpenVLA 需要的统一样本。
- 验证图像、语言、动作、dataset key 与时间对齐。
- 理解 RLDS 注册路径与 PyTorch wrapper 路径的取舍。

## 2. 今日知识点及其在 VLA 中的作用

adapter 是自有数据接入的边界；它必须保留来源与统计 key，并在训练前拒绝缺字段、错位帧和不可转换动作。

## 3. 必要概念、公式、形状与数据流

- 统一样本可抽象为 image、instruction、action、dataset_name、action_stats_key、metadata；真实字段名依所用代码版本核对。
- RLDS 路径通常需要 dataset config 与 transform；自定义 PyTorch Dataset 便于小样本验证，但训练循环、采样与复现责任更多。
- 动作监督采用 o_t → a_t；最后 step、无效 episode 与 timestamp 超容差样本要过滤并计数。

## 4. 分步骤学习内容

1. 定义 adapter 输入/输出 schema。
2. 实现 mock episode 到统一样本的 pure function。
3. 组 batch，打印 image/instruction/action 的 shape、dtype 与来源 id。
4. 注入错位 timestamp、缺语言、未知本体和末步 action 四类错误。

## 5. 必做作业

- 提交 custom_dataset_adapter.py、2 个 episode、batch summary 和至少 8 个测试。
- 写 RLDS 注册与 PyTorch wrapper 对比表。
- 输出过滤计数，禁止静默 drop。

## 6. 作业输入、预期输出与验证方法

| 项目 | 内容 |
|---|---|
| 输入 | 第 9 周 mock RLDS、action_contract.md 与本周 stats；无需 TFDS 或真实 OpenVLA。 |
| 预期输出 | 有效 step 形成 image batch 元数据 [B,3,H,W]、token 前文本列表 [B]、action [B,7]；过滤原因逐类计数。 |
| 验证方法 | 手工核对一个 episode 的 o_t,a_t。；shuffle 前后 sample id 与字段保持绑定。；删除 stats key，确认 adapter fail closed。 |

真实模型路径如需联网下载、安装或 GPU，必须先说明目的、磁盘/显存/时间成本并获得授权；默认完成低资源路径。

## 7. 提交内容与固定格式

```markdown
# W10D06 提交

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

硬门槛：过滤样本无计数、episode 边界被跨越，或 dataset key 未随样本传递，不得通过。

## 9. 常见错误与排查提示

- collate 时不同相机被错误 stack。
- 语言 episode-level 字段在 step 展开时丢失。
- 先 normalize 后又被 adapter 重复 normalize。
- 先打印输入 mode/shape、processor keys、model revision、action stats key 与 action shape，再排查数值。
- 不得声称执行过未实际运行的安装、推理或训练。

## 10. 时间不足时的最低完成线

完成 1 个 episode 到 3 个有效样本的转换、4 项 schema 断言和 2 个失败测试。 最低线也要提交实际证据或逐项“预期”标记。

## 11. 可选提高任务

加入 action chunk [B,H_a,7] 与 padding mask 支持。

## 12. 完成后的自测题

1. adapter 应在哪一步过滤最后 step？
2. 为什么 sample id 必须保留？
3. RLDS 与 PyTorch 路径各承担什么责任？

---

[← 上一日](day-05.md) · [本周首页](README.md) · [下一日 →](day-07.md)
