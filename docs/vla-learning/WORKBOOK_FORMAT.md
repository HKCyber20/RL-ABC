# VLA Notebook 作业格式说明

## 当前统一格式

根据学习者选择，12 周、84 天的正式作业全部使用 Jupyter Notebook。课程正文仍使用 Markdown，正式提交与验收以 notebooks 目录中的 ipynb 文件为准。

- 课程正文：docs/vla-learning/week-XX/day-XX.md
- 正式作业：docs/vla-learning/notebooks/week-XX/day-XX.ipynb
- 旧 Markdown 作业：docs/vla-learning/submissions/，仅作为迁移备份
- Goal 看板：docs/vla-learning/PROGRESS.md

## 每份 Notebook 的单元结构

1. Goal：当天目标与学习者复述；
2. Setup：时间、环境、硬件和资源等级；
3. Steps：课程必做题及每题独立 Markdown 作答单元；
4. Code：可运行代码或实验单元，理论日可留空；
5. Checks：shape、dtype、数值范围、正向与负向测试；
6. Evidence：命令、退出码、日志、配置、哈希和 VLA 约束；
7. Self-check：每道自测题的独立作答单元；
8. Help：常见错误、最低完成线和提高任务；
9. Rubric：当天评分标准；
10. Coach Review：教练验收区，学习者请勿填写；
11. Next Steps：保存路径和前后课程导航。

## 如何填写

1. 使用 JupyterLab、VS Code Notebook 或其他兼容编辑器打开文件；
2. 双击含有“请在这里填写”的 Markdown 单元格；
3. 删除占位文字，输入任意长度的文字、表格、公式、Mermaid 或说明；
4. 在 Code 单元中运行代码并保留必要输出；
5. 从上到下执行，避免依赖隐藏状态或乱序执行；
6. 保存 ipynb 后，将文件路径发到学习对话等待验收。

## 运行与证据规则

- Notebook 从上到下执行成功后，代码输出才可作为运行证据；
- 未执行内容必须明确标记为“预期结果”；
- 不要用预期输出冒充实际输出；
- 输出过长时保留关键摘要，不堆积完整调试日志；
- 涉及随机过程时记录 seed；
- 涉及机器人动作时记录坐标系、单位、动作逐维定义和时间对齐；
- 涉及成功率时同时记录成功次数、总次数和实验条件。

## 当前环境限制

2026-09-01 的检查结果显示，当前可见 Python 环境未检测到 Jupyter、nbformat、nbclient、ipykernel、PyTorch 或 NumPy。因此：

- 84 份 Notebook 已完成 JSON 和单元结构生成；
- 当前没有声称 Notebook 已执行；
- 没有擅自安装依赖；
- 真正开始代码实验前，需要准备可用的 Jupyter/Python 内核；
- 理论题可以先在兼容 Notebook 编辑器中填写 Markdown 单元。

## Day 1 迁移说明

学习者已经写入旧 Markdown Day 1 的目标复述、第 1 题定义和第 2 题闭环图，以上内容已迁移到 notebooks/week-01/day-01.ipynb。旧 Markdown 文件继续保留，避免任何学习成果丢失。
