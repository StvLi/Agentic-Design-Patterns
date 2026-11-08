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

## 补充：agent-as-tool 的两个实操问题

把子 agent 包成 `AgentTool` 后，常被问到的两件事（示例在 `(ADK_Gemini_AgentTool)` 的后两格）：

### ① 子 agent 怎么拿到"特定数据"（session.state）

**子 agent 与父 agent 共用同一个 session**，所以两种方式都可行：

| 方式 | 怎么写 | 机制 | 保真度 |
|---|---|---|---|
| **占位符** | 子 agent 的 `instruction` 里写 `{data}` | ADK 发请求前**直接替换** | ✅ 高（原值注入） |
| **读 state 工具** | 给子 agent 一个用 `ToolContext.state` 的函数工具 | 工具返回值 → **模型再写一遍** | ⚠️ 有失真风险 |

实测：占位符稳定拿到 `机密42`；工具方式出现过模型把 `机密42` 写成 `42`（丢前缀）。
**要精确传递数据，优先用占位符。**

### ② 主 agent 怎么消费子 agent 返回的字符串

子 agent 的最终文本，会作为 `function_response` 的 `result` 字段回到父 agent：

```python
function_response.name     # 子 agent 的名字（对父 agent 来说就是工具名）
function_response.response # {'result': '<子 agent 的最终文本>'}
```

父 agent 能不能"用好"它，取决于**父的 instruction 有没有写清怎么用**。实测：

```
[工具返回] SentimentProbe -> {'result': 'SENTIMENT=negative'}
[主 agent 最终答复] SENTIMENT=negative。对于这次延期给您带来的不便和失望，我们深表歉意；
                  建议您立即与项目负责人对接……
```

父 agent 既原样带上了 `SENTIMENT=negative`，又据此补了一句建议——这就是"消费"。
让子 agent 返回**机器可读的一行**（而不是一段散文），父 agent 才容易稳定使用。

## 模式：上层 agent 用 AgentTool 调度下层 LoopAgent

`(ADK_Gemini_Loop)` 最终定型成这个结构：

```
Supervisor（LlmAgent · 上层）
   └─ AgentTool(loop)          ← 把整个 LoopAgent 当工具调用
        └─ LoopAgent [Worker, ConditionChecker]
```

**为什么这样接**：上层 agent 不需要知道下层是"一个循环"，只知道"我有一个工具可调、调完有返回值"。
下层的复杂性（几轮、什么条件停、怎么停）完全封在工具后面——**这才是分层该有的样子**。

### 关键实现：让"结束那一轮"主动交出返回值

`AgentTool` 返回的是被包装 agent 的**最终文本**。而 LoopAgent 天然没有"最终文本"——
它结束时只是 escalate。所以要在 `ConditionChecker` 里把统计和退出信号**一起** yield：

```python
if s.get("status") == "completed":
    stats = f"轮次统计: 尝试={s.get('attempts')} 次; 最终状态=completed; 逐轮={s.get('log')}"
    yield Event(author=self.name,
                content=types.Content(role="model", parts=[types.Part(text=stats)]),
                actions=EventActions(escalate=True))     # 文本 + 退出信号
```

这段文本就成了 loop 的返回值，经 `function_response` 回到上层。

### 实测效果

```
[下层 loop 的返回值] {'result': "轮次统计: 尝试=3 次; 最终状态=completed;
                    逐轮=['try#1 NOT_DONE', 'try#2 NOT_DONE', 'try#3 DONE']"}
[上层 agent 的总结] 本次循环任务共跑了 3 轮。其中，前两轮的状态均为未完成（NOT_DONE），
                  在第三轮时顺利完成（DONE）。最终任务成功结束。
```

上层 agent 只做了一件它擅长的事——**把结构化统计写成自然语言**；
统计本身由代码算（确定性、不花 token）。

### 探索中的两个中间结论

1. **`AgentTool(agent=loop)` 直接可用，但返回值很"薄"**
   只返回 loop 里**最后一个说话的子 agent** 的文本（如 `'已完成'`），拿不到统计。
   所以必须让结束那一轮主动"说出"统计。

2. **还有一条等价写法**：`SequentialAgent(sub_agents=[loop, formatter])` 再包 AgentTool。
   让 formatter 读 state 占位符输出统计。多一个 agent，但不需要改 Checker。

### 三种对接方式（回顾本章的通信方式）

| 方式 | 上层怎么拿数据 | 整合度 | 代价 |
|---|---|---|---|
| **消息传递** | Python 拼 JSON 当 user message | 低 | 手工序列化、可能漏字段 |
| **共享 state** | `SequentialAgent` 串起来 + `{key}` 占位符 | 中 | 只能传 state 里有的；list 会变 repr |
| **AgentTool（推荐）** | 调工具 → `function_response` 返回 | **高** | 上层无需知道下层内部 |

## 六份脚本涉及的协作方式总结

### 一、逐个对应

| notebook | 书里的形态 | 核心机制 | 数据怎么传 | 契约强度 |
|---|---|---|---|---|
| `ADK_Gemini_Sequential` | **Sequential Handoffs** | `SequentialAgent` | `output_key` 写 state + `{key}` 占位符 | 弱（字符串名字） |
| `ADK_Gemini_Parallel` | **Parallel Processing** | `ParallelAgent` | 各写各的 state key，互不读 | 弱（字符串名字） |
| `ADK_Gemini_Coordinator` | **Hierarchical** | `sub_agents` + 子 agent 的 `description` | 整个对话转交 | 弱（自然语言） |
| `ADK_Gemini_AgentTool` | Hierarchical 变体（**agent-as-tool**） | `AgentTool(agent=...)` | `function_response` 返回子 agent 文本 | 弱（工具描述） |
| `ADK_Gemini_Loop` | 迭代控制流（书里六种没单列） | `LoopAgent` + `escalate` / `max_iterations` | 同上，state 记录进度 | — |
| `CrewAI_Gemini` | **Expert Teams** | `Agent(role/goal/backstory)` + `Task` + `Crew` | **`context=[上一个 task]`** | **强（对象引用）** |

### 二、覆盖度：书里列了 6 种，代码实现了哪几种

| 书里的形态 | 本章有代码吗 |
|---|---|
| Sequential Handoffs | ✅ `(Sequential)` |
| Parallel Processing | ✅ `(Parallel)` |
| Hierarchical | ✅ `(Coordinator)` + `(AgentTool)` 两种写法 |
| Expert Teams | ✅ `(CrewAI)` |
| Critic-Reviewer | ⚠️ 本章无脚本，但**第 4 章 Reflection 就是它**（生产者 + 批判者） |
| Debate & Consensus | ❌ 本章没有实现（需要多个 agent 同题作答再收敛） |

### 三、按"调用成本"排

| 形态 | 模型调用次数 | 说明 |
|---|---|---|
| CrewAI sequential | **2 次** | task 按顺序直跑，无中间协调者 |
| ADK Sequential / Parallel | **2 次** | 每个 agent 各 1 次 |
| ADK AgentTool | **2–3 次** | 父 agent 决策 + 子 agent 干活（+ 可能再组织一次回答） |
| ADK Coordinator | **3+ 次** | 协调者决策 1 次 + 子 agent 1 次 + 可能汇总 1 次 |
| ADK Loop | **2 × 轮数** | 每轮 worker + checker；条件不满足就继续烧 |

> 记住这条：**多 agent 的成本不是"加法"，是"上下文重发"**。
> ADK 转交时每次都会重发整个 prompt（官方 `context_cache_config` 警告说的就是这件事）。

### 四、选型速查

| 你要的效果 | 用哪个 |
|---|---|
| 固定顺序，A 的输出给 B | `SequentialAgent`（ADK）或 `Task(context=[...])`（CrewAI） |
| 几件事互不依赖、要快 | `ParallelAgent` |
| 让模型按内容动态选人 | `Coordinator`（`sub_agents` + description） |
| 上层要调用一个"专长"办件小事，拿回结果继续干 | `AgentTool` |
| 反复尝试直到成功 / 直到条件满足 | `LoopAgent`（`escalate` + `max_iterations`） |
| 角色分工做内容生产（研究员/作者/编辑） | CrewAI `Agent` + `Task` + `Crew` |
| 需要多个视角争论出结论 | ❌ 本章无代码，需自己搭 |

### 五、一条贯穿的判断

> **先问"它们之间要不要传数据"，再问"要不要让模型决定传给谁"。**
> - 要传数据 → 用 state / context（**别用自然语言转交**，那是给不需要数据的情况用的）
> - 要模型决定 → Coordinator；要写死 → Sequential/CrewAI
>
> 两个都要 → 先 state 传数据，外层再用 Coordinator 选人（组合起来）。

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
