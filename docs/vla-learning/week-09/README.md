# 第 9 周：论文、Open X-Embodiment 与 RLDS

[打开本周 Notebook 作业](../notebooks/week-09/README.md)

[← 第 8 周](../week-08/README.md) · [课程总入口](../README.md) · [第 10 周 →](../week-10/README.md)

## 本周核心能力与总 Goal 中的作用

**核心能力：**能从 RT-1、RT-2 与 Open X-Embodiment 中提取可复用的工程决策，并把异构机器人数据映射为可审计的 RLDS episode/step 与统一动作契约。

前八周得到的是一个单任务语言条件策略；本周补上通用 VLA 的数据组织、跨本体差异和论文比较能力，为第 10 周接入 OpenVLA 准备输入、动作与归一化契约。

## 前置知识

- 能解释 `图像 + 指令 + 本体状态 → 动作` 的闭环数据流。
- 会用 Python/PyTorch 或纯 Python 检查字典、数组形状与数值范围。
- 已经理解行为克隆、相对末端动作、成功率和训练/验证划分。

## 推荐一手资料

- [RT-1 项目页](https://robotics-transformer1.github.io/)
- [RT-2 项目页](https://robotics-transformer2.github.io/)
- [Open X-Embodiment / RT-X 项目页](https://robotics-transformer-x.github.io/)
- [RLDS 官方仓库](https://github.com/google-research/rlds)

阅读时记录访问日期或 commit；课程中的结构化练习不要求下载全量数据。

## 7 天导航

| 日程 | 课程 | 当天可验证结果 |
|---|---|---|
| Day 1 | [RT-1：从视觉语言表示到动作 token](day-01.md) | 说明 RT-1 如何把图像、语言和动作统一进序列建模问题。 |
| Day 2 | [RT-2：把 VLM 知识迁移到机器人动作](day-02.md) | 解释 RT-2 中“动作也作为 token 序列”的核心思想。 |
| Day 3 | [Open X-Embodiment：异构数据混合的机会与代价](day-03.md) | 理解 OXE 解决的数据规模与多样性问题。 |
| Day 4 | [RLDS：episode/step schema 与时间语义](day-04.md) | 准确解释 RLDS 的 episode 与 step 层级。 |
| Day 5 | [跨本体动作对齐与归一化契约](day-05.md) | 区分原始动作、统一语义动作、归一化动作和部署动作。 |
| Day 6 | [低资源 RLDS mock inspector](day-06.md) | 把本周 schema、时序与动作契约实现为自动检查器。 |
| Day 7 | [论文对比与周项目验收](day-07.md) | 把 RT-1、RT-2、OXE/RT-X 的关键决策放进同一比较框架。 |

## 本周代码与实验产物

- `w09_paper_matrix.md`：RT-1、RT-2、OXE/RT-X 与后续 OpenVLA 的结构化比较。
- `mock_rlds_episode.json`：至少 2 个 episode 的小型可读样例。
- `inspect_rlds_mock.py` 与运行日志：检查 schema、时间对齐、形状、有限值和边界标志。
- `action_contract.md`：本体原始动作到统一动作的转换与反归一化契约。
- `w09_week_project.md`：数据审计结论、失败样例与修复证据。

## 必做任务

- 完成 7 个日课且每份提交达到 80 分。
- 周项目必须包含一个故意损坏的 episode，检查器应拒绝它并给出可定位错误。
- 能够口头解释 `is_last` 步为何不能被当作普通监督样本。
- 所有“实测结果”必须附命令或输出证据；未运行内容只能标为“预期”。

## 可选任务

- 在不下载全量数据的前提下，阅读一个公开 OXE dataset card 并补充字段映射。
- 为 mock inspector 增加 JSON Schema 或 property-based test。
- 比较 percentile normalization 与标准差归一化在异常值下的差异。

## 时间不足时的最低完成线

只使用 CPU 和 Python 标准库完成 2 个 mock episode、一个 schema/时序检查器、动作转换契约及论文比较表；不要求 TensorFlow、TFDS、真实 OXE 数据或 GPU。

## 周末概念测验

1. 解释 RT-1 与 RT-2 的动作输出思路有何共同点和差异。
2. 画出 RLDS `episode → steps → observation/action` 层级，并解释 `is_first/is_last/is_terminal`。
3. 给定两个本体的动作定义，指出为什么不能直接拼接训练。
4. 说明归一化统计量必须如何与 dataset key、动作维度和反归一化绑定。

闭卷作答后再核对资料；80 分通过。凡涉及动作 frame、末步语义或归一化 key 的关键题答错，需完成补救题后再验收。

## 周项目

对两个自建 mock embodiment 做一次“跨本体数据接入审计”：定义原始 schema，转换到统一 7 维动作，验证 step 边界与 `o_t → a_t → o_{t+1}` 对齐，生成通过/拒绝报告，并用论文比较矩阵说明这些设计分别回应了哪些通用 VLA 难题。

## 本周通过标准

- 7 个日课均达到 80/100，且无硬门槛失败。
- 检查器能接受合法样例、拒绝至少 3 类错误：末步动作被使用、形状不一致、NaN/Inf 或边界标志错误。
- 统一动作的单位、坐标系、符号、维度、归一化统计和反归一化流程均有书面定义。
- 周项目可在无网络、无 GPU 环境由一条明确命令复跑；若未实际复跑，必须标注为“预期”，不能宣称通过。

进度只能在教练依据提交证据验收后更新，学习者自行阅读或口头声明不等于通过。

---

[← 第 8 周](../week-08/README.md) · [课程总入口](../README.md) · [第 10 周 →](../week-10/README.md)
