# 第 12 周 Day 2：配置、随机种子与环境复现

[填写本日 Notebook 作业](../notebooks/week-12/day-02.ipynb)

[← 上一日](day-01.md) · [本周首页](README.md) · [下一日 →](day-03.md)

## 1. 今日目标与预计时间

预计：**120–170 分钟**。

- 建立单一配置入口与 seed 传播表。
- 记录依赖、硬件、代码版本和产物目录。
- 实现从干净进程运行的 smoke test。

## 2. 今日知识点及其在 VLA 中的作用

复现失败往往来自隐含默认值、路径和随机性；配置与环境证据让训练和评估结果可追踪。

## 3. 必要概念、公式、形状与数据流

- 配置至少含 data/split、model、optimizer、seed、train/eval、action stats、output 与 code revision；CLI override 要记录最终解析值。
- seed 传播到 Python、NumPy、PyTorch、环境 reset、数据采样和评估初态；确定性仍可能受算子/并行影响，需记录而非保证绝对一致。
- 目录按 run_id 保存 config snapshot、environment、logs、checkpoint、manifest、metrics，禁止覆盖历史 run。

## 4. 分步骤学习内容

1. 整理一份最终 config schema。
2. 实现 set_seed 并输出 seed trace。
3. 生成 environment lock 与硬件信息采集命令；未执行项标未知。
4. 从新进程执行最小 reset→observe→act→step smoke test。

## 5. 必做作业

- 提交 configs/final.yaml、ENVIRONMENT.md、REPRODUCE.md 草稿与 smoke log。
- 用相同 seed 两次运行 smoke，比较关键状态；再换 seed 证明初态变化。
- 测试缺少 stats key/spec hash 时启动失败。

## 6. 作业输入、预期输出与验证方法

| 项目 | 内容 |
|---|---|
| 输入 | Day 1 spec 与项目环境；无重型仿真时使用 CPU 轻量环境。 |
| 预期输出 | 一条命令启动 smoke；最终解析配置被保存；同 seed 关键 trace 一致或差异被解释。 |
| 验证方法 | 从新 shell/进程运行。；检查输出目录无硬编码个人临时路径。；比较 config hash 与 manifest 记录。 |

本周“预期”不能替代最终核心闭环实测；需要大模型/GPU 的 OpenVLA 部分可用明确标注的 CPU/mock dry-run。

## 7. 提交内容与固定格式

```markdown
# W12D02 提交

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

硬门槛：seed 只设置 PyTorch 未设置环境，配置缺 spec/stats key，或运行覆盖旧结果，不得通过。

## 9. 常见错误与排查提示

- 修改 config 后仍复用旧 hash。
- 依赖只写 Python 版本。
- 相对路径依赖当前工作目录却未说明。
- 先核对 spec/config/data/checkpoint/result hash、seed、stats key、动作单位/frame，再解释性能。
- 不得把手写 outcome、mock OpenVLA 输出或预期命令包装成真实闭环/模型结果。

## 10. 时间不足时的最低完成线

一个 final config、一条 smoke 命令、两个 seed trace 和环境说明。 最低完成线仍须满足当天硬门槛。

## 11. 可选提高任务

加入 CI smoke job 或容器文件，但不未经授权下载镜像。

## 12. 完成后的自测题

1. 为什么同 seed 仍可能有微差？
2. run_id 应包含什么？
3. 怎样防止旧 checkpoint 配新 stats？

---

[← 上一日](day-01.md) · [本周首页](README.md) · [下一日 →](day-03.md)
