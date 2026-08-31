# 第 6 周 Day 1：视觉预处理与编码器

[填写本日 Notebook 作业](../notebooks/week-06/day-01.ipynb)

[上一课：第 5 周 Day 7](../week-05/day-07.md) · [本周首页](README.md) · [下一课：Day 2](day-02.md)

## 1. 今日目标与预计时间

预计 120 分钟。把磁盘 RGB 稳定转换为模型张量，并用 tiny CNN 在 CPU 上完成 forward/backward、shape 和数值范围检查。

## 2. 这在 VLA 中的作用

视觉 encoder 把像素压缩成任务相关特征。若 RGB/BGR、HWC/CHW、缩放或 normalization 出错，后面的融合与控制即使代码无异常也会学习错误分布。

## 3. 核心概念、公式与张量

预处理数据流：

```text
uint8 RGB [B,H,W,3]
→ float32 / 255
→ resize/crop
→ permute [B,3,H,W]
→ channel normalize
→ CNN → z_img [B,D]
```

通道归一化：`x'=(x-μ)/σ`。tiny CNN 可采用三层 stride-2 卷积 + global average pooling，避免依赖下载；例如输入 `[B,3,64,64]` 输出 `[B,128]`。

## 4. 分步学习

1. 从第 5 周随机加载 8 帧，检查 dtype、范围、通道和非空内容。
2. 实现单一 `preprocess(rgb)`，避免训练和评估各写一套。
3. 保存预处理前后可视化，反归一化后确认颜色合理。
4. 实现 tiny CNN 或使用本机已有 encoder，不联网下载权重。
5. 对 `[2,3,64,64]` 做 forward/backward，打印特征 shape、均值、标准差和首层梯度范数。

## 5. 必做作业

- 写 6 个预处理断言：输入 shape/dtype、输出 shape/dtype、有限值、范围或 normalization、可逆可视化。
- 实现 `VisualEncoder(output_dim=128)`，参数量可统计，输出必须固定维度。
- 运行一次相同输入的 eval 模式确定性测试，以及一次 backward 梯度测试。

## 6. 输入、预期输出与验证

输入：8 张真实或 synthetic `64×64 RGB uint8`；batch size 2。没有数据时生成红/蓝方块位置随 state 变化的图像，不能使用与动作无关的纯随机噪声。

预期输出：预处理张量 `[2,3,64,64] float32`；encoder 输出 `[2,128]`；loss.backward 后至少一个 encoder 参数梯度有限且非零。均为预期结果。

验证：固定 seed 和 `model.eval()` 时两次输出 `allclose`；故意交换通道或输入错误 dtype 时断言失败；反归一化图像与原图平均误差在量化容差内。

## 7. 固定提交格式

```markdown
# W6D1 提交
- 实际用时：
## 原始数据检查
## 预处理公式与 shape 流
## Encoder 结构与参数量
## 正向/反向运行命令及日志
## 负向测试与可视化
## 自测答案
```

## 8. 评分与验收

- 数据检查与单一预处理实现：20 分。
- shape/数值断言完整：20 分。
- encoder forward 输出正确：20 分。
- 确定性和 backward 证据：25 分。
- 负向测试、自测和低资源说明：15 分。

总分 100，80 分通过；通道顺序不明、无 backward 证据或 synthetic 图像与任务无关时不通过。

## 9. 常见错误与排查

| 错误 | 排查 |
|---|---|
| uint8 直接相减溢出 | 先转 float32 再缩放 |
| normalize 两次 | 让 Dataset/模型只有一个明确责任边界 |
| BatchNorm 小 batch 不稳定 | CPU 小样本用 GroupNorm/LayerNorm 或固定 eval |
| flatten 依赖分辨率 | 用 adaptive/global pooling |

## 10. 时间不足时的最低完成线

用 8 张 synthetic 任务图像和 CPU tiny CNN 完成 6 个断言、`[2,128]` 输出与非零梯度测试。

## 11. 可选提高

比较 global pooling 与 spatial tokens；测 encoder 吞吐；可视化第一层 feature maps。

## 12. 完成后的自测题

1. 预处理为何必须在训练和 rollout 共用？
2. 输出 feature 是 `[B,D]` 与 `[B,N,D]` 各适合什么融合？
3. eval 模式确定性测试能发现哪些问题？
4. synthetic 图像为什么必须与动作目标相关？

[上一课：第 5 周 Day 7](../week-05/day-07.md) · [本周首页](README.md) · [下一课：Day 2](day-02.md)
