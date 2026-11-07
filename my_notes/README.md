# 我的学习笔记

《Agentic Design Patterns》逐章笔记。每章一份，配合 `chapter_notebooks/` 里的注释版
notebook（已适配本机：DeepSeek / 中转站 Gemini / 本地 Qwen）。

## 索引

| 章节 | 笔记 | 一句话 |
|---|---|---|
| 第 1 章 | [ch01_prompt_chaining.md](ch01_prompt_chaining.md) | 链只提升可控性，不提升正确性 |
| 第 2 章 | [ch02_routing.md](ch02_routing.md) | 改一句自然语言，分类边界就平移，且没有 diff 可看 |
| 第 3 章 | [ch03_parallelization.md](ch03_parallelization.md) | 并行只对 I/O 有效；契约缺 key 是硬崩不是降级 |
| 第 4 章 | [ch04_reflection.md](ch04_reflection.md) | 收敛取决于生成者能力，不只是批判者判断力 |
| 第 4 章 · 实验记录 | [ch04_experiment_transcript.md](ch04_experiment_transcript.md) | 四组"生成者 × 批判者"对照的完整记录 |
| 第 5 章 | [ch05_tool_use.md](ch05_tool_use.md) | 工具的三种来源；越内建越难观测 |
| 第 6 章 | [ch06_planning.md](ch06_planning.md) | 计划要成为数据，而不是散文 |
| 第 7 章 | [ch07_multi_agent.md](ch07_multi_agent.md) | 三种通信方式：共享 state / 转交 / 当成工具调 |
| 第 7 章 · 补充笔记 | [ch07_notes_communication.md](ch07_notes_communication.md) | 顺序靠串行 await；并行时撞 state key 就是随机 bug |

## 规范与工具文档

| 文档 | 内容 |
|---|---|
| [annotation_convention.md](annotation_convention.md) | 注释规范：`★ 本章新增` / `（ChN）` / `【适配】`，以及"三种载体各放什么" |
| [ros_multiagent_protocol_notes.md](ros_multiagent_protocol_notes.md) | 设计备忘：用 ROS/ROS2 做多 agent 通信中间件（初步想法，待验证） |
| [ROADMAP.md](ROADMAP.md) | 学习路线图：全书进度、各章产出、环境与踩坑记录 |
| [labs_study_plan.md](labs_study_plan.md) | 实验（lab）学习计划：顺序、每节内容、精华路线 |

## 阅读顺序建议

**按章顺序读**。前三章是"原子操作"：串行（Ch1）、分支（Ch2）、并发（Ch3）。
第 4–5 章补上"自我改进"和"能动手"。第 6–7 章把它们编起来：先规划，再多 agent 协作。

如果只有很短的碎片时间，优先读每章笔记末尾的**"一句话"**和**"陷阱"**两节。

## 目录约定

```
Agentic-Design-Patterns/
├── my_notes/                 ← 这个目录：所有学习笔记（本仓库版本控制）
├── chapter_notebooks/        ← 书的原 notebook（已加中文注释 + 本机适配）
```

仓库外的同级目录（不进版本控制）：

```
../ch01_prompt_chaining/      ← 各章的可跑脚本与调试实验室
../common/                    ← 共享代码：make_llm() / adk_model() 等
../docs/google_credentials.md ← 凭证配置说明（含占位符，无真实密钥）
../.env                       ← 密钥（0600，已被 gitignore）
```
