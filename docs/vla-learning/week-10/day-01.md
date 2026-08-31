# 第 10 周 Day 1：OpenVLA 架构与端到端数据流

[填写本日 Notebook 作业](../notebooks/week-10/day-01.ipynb)

[← 上一日](../week-09/day-07.md) · [本周首页](README.md) · [下一日 →](day-02.md)

## 1. 今日目标与预计时间

预计：**90–120 分钟**。

- 识别 OpenVLA 中视觉编码器、投影/融合、语言模型、action tokenizer/预测接口的职责。
- 画出训练与推理两条数据流并标出差异。
- 区分稳定概念和版本相关实现。

## 2. 今日知识点及其在 VLA 中的作用

架构图决定调试边界：图像/提示词错在 processor，动作 token 错在模型/解码，物理范围错在反归一化或 controller，不能把所有问题统称为“模型不行”。

## 3. 必要概念、公式、形状与数据流

- 经典 OpenVLA 路径建立在 Prismatic VLM 之上，视觉表征、语言骨干与动作预测形成端到端策略；具体 backbone、脚本参数和新配方应以所用官方 commit 为准。
- 推理抽象：image PIL/RGB + prompt → processor tensors → VLA.predict_action → normalized/canonical action [B,D_a] → dataset-specific unnormalize → physical action。
- 训练标签可以是离散 action token；较新的优化配方可能采用连续动作与 action chunk。课程分别讨论接口，不假设它们可直接互换。

## 4. 分步骤学习内容

1. 阅读官方仓库首页/模型卡的架构段，记录访问日期或 commit；无网络则标为待核对，不写成实测。
2. 画模块图，给每条边写对象类型或 shape。
3. 建立“模块—可观察输入—输出—常见故障—检查方法”表。
4. 分别说明离散单步动作与连续 action chunk 对闭环频率的影响。

## 5. 必做作业

- 提交 openvla_architecture.md。
- 画训练数据流与推理数据流各一张。
- 列出至少 6 个版本相关项，说明为何要锁 commit/依赖。

## 6. 作业输入、预期输出与验证方法

| 项目 | 内容 |
|---|---|
| 输入 | 第 9 周 adapter handoff；可选查阅官方 OpenVLA 仓库，不要求 clone 或下载 checkpoint。 |
| 预期输出 | 图中 image、instruction、model output、unnorm key、stats 与 controller 相连；训练流含 label/loss，推理流含闭环环境反馈。 |
| 验证方法 | 逐边核对数据类型和 shape。；检查 action stats 不在语言模型内部被默认猜测。；用一个故障例说明应在哪层定位。 |

真实模型路径如需联网下载、安装或 GPU，必须先说明目的、磁盘/显存/时间成本并获得授权；默认完成低资源路径。

## 7. 提交内容与固定格式

```markdown
# W10D01 提交

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

硬门槛：漏掉 dataset-specific 反归一化或把新旧动作配方描述为完全相同接口，不得通过。

## 9. 常见错误与排查提示

- 把模型生成的 token ID 直接当米/弧度。
- 只画模型，不画 processor 与 controller。
- 引用最新版特性却未记录版本来源。
- 先打印输入 mode/shape、processor keys、model revision、action stats key 与 action shape，再排查数值。
- 不得声称执行过未实际运行的安装、推理或训练。

## 10. 时间不足时的最低完成线

完成一张 6 模块架构图、训练/推理差异表和 3 个故障定位例。 最低线也要提交实际证据或逐项“预期”标记。

## 11. 可选提高任务

比较原始 OpenVLA action token 路径与 OFT/FAST 的目标、输出形态，只陈述可核对信息。

## 12. 完成后的自测题

1. 哪个模块决定图像张量格式？
2. 动作物理尺度从哪里来？
3. action chunk 如何改变控制循环？

---

[← 上一日](../week-09/day-07.md) · [本周首页](README.md) · [下一日 →](day-02.md)
