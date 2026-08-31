# 第 10 周 Day 3：推理接口：processor、prompt 与 predict_action

[填写本日 Notebook 作业](../notebooks/week-10/day-03.ipynb)

[← 上一日](day-02.md) · [本周首页](README.md) · [下一日 →](day-04.md)

## 1. 今日目标与预计时间

预计：**110–150 分钟**。

- 定义可测试的推理函数签名与异常行为。
- 用 mock processor/model 复现 OpenVLA 风格接口。
- 记录确定性、设备和 dtype 契约。

## 2. 今日知识点及其在 VLA 中的作用

真实模型很重，但接口错误可用轻量 mock 提前发现；这使下载 checkpoint 前就能验证样本、提示词、unnorm key 与输出形状。

## 3. 必要概念、公式、形状与数据流

- 稳定抽象可写为 predict(image, instruction, unnorm_key) -> action[D_a]；具体类名/参数需以所用版本为准。
- processor 通常把 prompt 与 PIL image 转为 token/像素张量；模型输出动作时需明确 do_sample、dtype、device 和 unnorm key。
- mock 只能证明控制流与契约，不能证明视觉理解或动作质量；输出必须标注 source=mock。

## 4. 分步骤学习内容

1. 定义 InferenceRequest/InferenceResult 数据结构。
2. 实现 mock processor：拒绝非 RGB、空指令，返回固定 shape 元数据。
3. 实现 mock model：根据固定 seed 返回 normalized action，并校验 unnorm key。
4. 输出逐层 trace，包含 input hash、shape、dtype、key、action source。

## 5. 必做作业

- 提交 mock_openvla_inference.py 和至少 5 个测试。
- 测试合法请求、灰度图、空指令、未知 key、错误 action dim。
- 写真实模型替换点，不复制未经版本核对的完整命令。

## 6. 作业输入、预期输出与验证方法

| 项目 | 内容 |
|---|---|
| 输入 | 两张 64×64 RGB mock 图元数据、两条指令、unnorm_key=mock_arm_v1、动作维 7。 |
| 预期输出 | 合法请求返回 [7] 且 source=mock；四类坏输入给出稳定错误码；trace 不包含“真实模型成功”措辞。 |
| 验证方法 | 运行/预期测试逐项记录。；固定 seed 后复跑输出一致。；检查报告确认每个动作都携带 unnorm key。 |

真实模型路径如需联网下载、安装或 GPU，必须先说明目的、磁盘/显存/时间成本并获得授权；默认完成低资源路径。

## 7. 提交内容与固定格式

```markdown
# W10D03 提交

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

硬门槛：mock 输出被描述为模型推理结果，或函数允许未知 unnorm key 静默回退，不得通过。

## 9. 常见错误与排查提示

- processor 输出 batch 维被丢失。
- 在接口内部硬编码一个默认机器人统计。
- 异常被 catch 后返回全零动作而不报警。
- 先打印输入 mode/shape、processor keys、model revision、action stats key 与 action shape，再排查数值。
- 不得声称执行过未实际运行的安装、推理或训练。

## 10. 时间不足时的最低完成线

完成一个合法、三个非法请求和结构化 trace。 最低线也要提交实际证据或逐项“预期”标记。

## 11. 可选提高任务

加入超时、请求 id 和 REST 层序列化测试，不启动真实服务。

## 12. 完成后的自测题

1. processor 与 model 各负责什么？
2. 为什么错误时返回全零也可能危险？
3. 确定性设置记录哪些字段？

---

[← 上一日](day-02.md) · [本周首页](README.md) · [下一日 →](day-04.md)
