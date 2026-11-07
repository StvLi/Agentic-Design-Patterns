# 第 7 章 · Multi-Agent Collaboration

书里给了 **6 个脚本**，全在 `chapter_notebooks/Chapter_07_Multi_Agent_(...).ipynb`。
这一章的代码量比前面几章大，但**概念只有一个**。

## 一句话

> 单个 agent 有天花板。把它拆成几个**专职** agent，让它们协作。

书里的判断标准：

> 当一个任务对单个 agent 来说太复杂、**且能拆成需要不同技能/工具的子任务**时，才用这个模式。

听起来像废话，但这条判断有反面：**单 agent 够用就别拆**。拆分的代价见下面「陷阱」一节。

## 书里列的六种协作形态

| 形态 | 什么意思 |
|---|---|
| **Sequential Handoffs** | A 做完交给 B，像流水线 |
| **Parallel Processing** | 几个 agent 同时干不同的部分，最后合并 |
| **Debate & Consensus** | 多个 agent 从不同视角讨论，收敛出结论 |
| **Hierarchical** | 一个 manager 按能力动态分派任务给 worker |
| **Expert Teams** | 不同领域专家（研究员/作者/编辑）合作产出 |
| **Critic-Reviewer** | 一批人产出，另一批人评审，再返工（第 4 章的多人版） |

## 六个脚本分别演示哪一种

| notebook | 形态 | 关键机制 |
|---|---|---|
| `(ADK_Gemini_Sequential)` | Sequential Handoffs | `SequentialAgent` + `output_key` 传数据 |
| `(ADK_Gemini_Parallel)` | Parallel Processing | `ParallelAgent`，各写各的 state key |
| `(ADK_Gemini_Coordinator)` | Hierarchical | 父 agent + `sub_agents`，模型自己决定转交给谁 |
| `(ADK_Gemini_Loop)` | 迭代轮询（不在上面六种里） | `LoopAgent` + 自定义 agent 检查 state 决定是否停 |
| `(ADK_Gemini_AgentTool)` | Hierarchical 的变体 | 把**一个 agent 包装成工具**给另一个 agent 调 |
| `(CrewAI_Gemini)` | Expert Teams | `Agent`(角色) + `Task`(context 串联) + `Crew` |

## 三种"通信方式"（这才是本章的技术核心）

多 agent 的关键不是"有几个 agent"，而是**它们之间怎么传数据**。这六份代码其实只用了三种：

| 方式 | 怎么传 | 出现在 |
|---|---|---|
| **共享 state** | A 写 `output_key="x"`，B 在 instruction 里读 `{x}` | Sequential / Parallel / Loop |
| **转交（handoff）** | 父 agent 直接把对话交给子 agent | Coordinator |
| **当成工具调** | 子 agent 包装成 `AgentTool`，父 agent 像调函数一样调它 | AgentTool |

书里那句话值得记住：*"体系的效能不只来自分工，还取决于 agent 之间的通信机制。"*

而**这三种方式全都靠字符串名字维系**：
`output_key` 的键名、子 agent 的 `name`、`AgentTool(description=...)` 的描述。
写错了不会编译报错——这就是第 2、3 章反复出现的那个老问题，在多 agent 里被放大了。

## 陷阱（本章最该记住的部分）

1. **多 agent ≠ 更好。**
   每次转交都是一次新的模型调用：**成本和延迟成倍放大**。
   第 5 章我们实测过：ADK 一次转交就是 2~3 次 LLM 调用。

2. **错误会传播，而且没人喊疼。**
   单 agent 错了你还能看它的输出；多 agent 里 A 的错误会变成 B 的输入，
   最后你看到的是一个"看起来很合理"的结果。定位要到每一步的 state 里翻。

3. **协调者是瓶颈。**
   Hierarchical 里所有决策都过 manager，它的上下文会长得最快、也最容易成为单点。

4. **调试难度指数上升。**
   N 个 agent 就有 N 份 prompt、N 个 state 槽、N 套失败模式。

5. **书里几个例子是片段，跑不起来**（这一章的普遍情况）：
   - `Sequential`：`Agent(name="Step1_Fetch")` **没给 model**，构造就会失败
   - `Parallel`：给了 model，但 runner 只是写在注释里的"概念示意"
   - `Loop`：instruction 让模型"把 status 设成 completed"——**但 LlmAgent 不能直接写 state**
   - `Coordinator` / `AgentTool`：只有模型名过时，结构是完整的

   本仓库已把它们补成可运行版本，改动都用 `【适配】` 标注。

## 一条 API 现状：这些容器 agent 正在被 `Workflow` 取代

跑 `Sequential` / `Parallel` / `Loop` 时会看到：

```
DeprecationWarning: SequentialAgent is deprecated in favor of Workflow
ParallelAgent / LoopAgent 同理
```

官方建议迁移到 `google.adk.workflow.Workflow`（graph-based）。但书里用的这套**目前仍能正常工作**，
而且 `Workflow` 还不能当 `LlmAgent` 的子 agent，所以本章保留原写法。
这和第 3 章 ADK 版记录的是同一条弃用通知。

## 什么时候**不**该用

- 任务拆不开（就是一件事）
- 拆了之后 agent 之间要来回传大量上下文（通信成本 > 分工收益）
- 你要的是可预测性（多 agent 的不确定性远高于单 agent）

> 一句话：**先证明单 agent 不够，再拆。**

## 本机环境

| notebook | 内核 |
|---|---|
| 5 份 ADK | `Python (agentic-adk)` |
| 1 份 CrewAI | `Python (agentic-crewai)` |

ADK 侧统一用 `common.adk_model()`（官方 / 中转站可切），CrewAI 侧用内置 `deepseek` provider。
