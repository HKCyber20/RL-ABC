# 第 3 周 Day 2：从轨迹构造 BC Dataset

[填写本日 Notebook 作业](../notebooks/week-03/day-02.ipynb)

[← 上日：专家示范](day-01.md) · [周首页](README.md) · [次日：损失与归一化 →](day-03.md)

## 1. 今日目标与预计时间

预计 90–120 分钟。将多个变长 episode 展开为监督样本，正确实现索引映射、batch 拼接和数据验证，不跨越 episode 边界。

## 2. 今日知识点及其在 VLA 中的作用

行为克隆把专家数据视作监督学习：`x_t=(o_t,l,s_t)`，标签为 `a_t`。Dataset 看似简单，却最易引入时间错位、episode 边界穿越、dtype/shape 混乱。可靠数据加载器比先换更大模型更重要。

## 3. 必要概念、公式、形状与数据流

对 episode `e` 长度 `T_e`，单步数据集总样本数：

\[
N=\sum_e T_e
\]

索引表可显式存 `index -> (episode_id,t)`。batch 示例：

```text
state  [B,d_s]
action [B,d_a]
goal/language feature [B,d_l]  # 本周可用目标位置代替文本
```

未来含图像时 `image [B,C,H,W]`，但今天用低维状态确保数据逻辑可查。

## 4. 分步骤学习

1. 构造长度 3、5、4 的三个 episode。
2. 建立全局 index map 并手查首尾项。
3. 实现 `__len__`、`__getitem__` 和 batch collate。
4. 遍历所有样本，验证 shape/dtype/finite 和 episode/t。
5. 统计每个 episode 贡献样本数，防止遗漏或重复。

## 5. 必做作业

实现 `BCDataset`，输入 episode 列表，返回 `state,goal,action,episode_id,t`。使用长度 3/5/4 的固定结构测试，总样本应为 12。提交全局索引 0、2、3、7、8、11 的映射；用 batch size 4 迭代并打印形状。再写一个会跨 episode 取 `t+1` 的错误版本片段并解释如何检测。

## 6. 输入、预期输出与验证方法

- 输入：3 个 episode，`d_s=2,d_goal=2,d_a=2`。
- 预期：len=12；完整 batch `[4,2]`；索引映射在 episode 边界正确。
- 验证：收集 `(episode_id,t)` 应恰好覆盖每个合法 pair 一次；动作应等于原 episode 的 `action[t]`；所有张量有限且浮点 dtype 一致。

低资源替代：用 Python 列表或表格完成 index map，无需 PyTorch DataLoader。

## 7. 提交内容与固定格式

固定包含：`数据接口`、`索引映射`、`Dataset代码/伪代码`、`batch日志`、`覆盖性验证`、`边界错误案例`、`投入分钟数`。

## 8. 评分与验收标准

100 分，80 分通过：索引 25；实现 25；形状/dtype 15；覆盖验证 20；边界分析 15。跨 episode、len 错误或 action 对齐错误任一为关键失败。

## 9. 常见错误与排查

- 用最大 T 乘 episode 数：变长数据会产生伪样本。
- `__getitem__` 返回 NumPy/torch 混杂：collate 时明确转换。
- batch shuffle 后误判时间错位：保留 episode_id,t 用于审计。
- 把终态作为有动作标签的样本：单步 BC 通常不包括 `state[T]`。

## 10. 时间不足时的最低完成线

完成 12 项 index map、首尾/边界断言和一个 batch 形状。

## 11. 可选提高任务

加入样本权重，使每个 episode 总权重相等，比较与按 transition 均匀采样的差异。

## 12. 完成后的自测题

1. Dataset 总长度为什么不是状态总数？
2. 保留 episode_id,t 对调试有什么价值？
3. 变长 episode 如何避免 padding？何时才需要 padding？
4. shuffle 会破坏单步 BC 吗？为什么？

[← 上日：专家示范](day-01.md) · [周首页](README.md) · [次日：损失与归一化 →](day-03.md)
