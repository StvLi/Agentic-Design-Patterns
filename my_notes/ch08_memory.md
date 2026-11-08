# 第 8 章 · Memory Management

书里给了 **5 份脚本**，全在 `chapter_notebooks/Chapter_08_Memory_(...).ipynb`。
这一章的代码都很短，**概念比代码重要**。

## 一句话

> 给 agent 加"记住"的能力。分两层：**短期**（一次对话内）和**长期**（跨对话、可检索）。

书里的判断标准：

> 当 agent 要做的不止"回答单个问题"时，就需要记忆。
> 要维持对话上下文、要跨多步跟踪进度、要记住用户偏好——都属于这一类。

## 三个概念，必须先分清

| | 是什么 | 生命周期 | ADK 里的对象 |
|---|---|---|---|
| **Session** | 一次对话的线程 | 一次会话 | `SessionService` |
| **State** | 这次会话里的临时数据（键值） | 随 Session | `session.state` / `output_key` |
| **Memory** | 跨会话、可语义检索的长期知识库 | 永久 | `MemoryService` |

一句话记：**Session 是"这次聊了什么"，State 是"这次记住什么"，Memory 是"以前学到什么"。**

`MemoryService` 的接口只有两个方向：

- **写**：`add_session_to_memory(session)` —— 把一次会话的内容**抽取**进长期库
- **读**：`search_memory(app_name, user_id, query)` —— 语义检索

## 三种长期记忆（对人的类比）

| 类型 | 记什么 | 典型实现 |
|---|---|---|
| **语义记忆** | 事实、偏好（"用户喜欢靠窗座位"） | 用户 profile（JSON）或事实文档集合 |
| **情节记忆** | 经历过什么、怎么做成的 | **few-shot 示例**：把过去成功的交互序列当例子 |
| **程序记忆** | 怎么做事的规则（系统提示本身） | **Reflection**：让 agent 读自己的指令 + 近期交互，自己改写指令 |

第三种最能体现"记忆真的改变行为"——脚本 5 的最后一格就是它。

## 五份脚本各演示什么

| notebook | 演示 | 能跑吗 |
|---|---|---|
| `(ADK_Explicit_State_Update)` | **用工具**显式改 `state`（登录事件） | ✅ |
| `(ADK_LlmAgent_output_key)` | `output_key` 把最终答复自动写进 `state` | ✅ |
| `(ADK_MemoryService_InMemory)` | 长期记忆服务：InMemory 实现 + 真做一次"写进去再检索" | ✅ |
| `(ADK_SessionService)` | 三种会话后端：内存 / SQLite / Vertex AI | ✅（Vertex 需 GCP） |
| `(LangChain_LangGraph)` | LangChain 的记忆演进 + LangGraph Store 存"指令" | ✅ |

## 本章最实用的一条：**写 state 只有三条路**

这是第 7 章 Loop 那个坑的直接延续：

| 方式 | 谁写 | 例子 |
|---|---|---|
| **`output_key="x"`** | 框架自动（写这个 agent 的最终答复） | 脚本 2 |
| **工具里的 `tool_context.state[...]`** | 你的代码 | 脚本 1 |
| **`Event(actions=EventActions(state_delta={...}))`** | 自定义 agent（`BaseAgent`） | 第 7 章 Loop |

**唯独 `LlmAgent` 自己写不了 state**——它的 instruction 里说"把 status 设成 completed"是空话。
而且 `context.session.state` 在自定义 agent 里是**只读**的（读得到、写不进），要写必须发带 `state_delta` 的事件。

## 陷阱

1. **短期记忆受上下文窗口限制**：对话越长越贵，且会被截断。要么裁剪、要么摘要、要么转长期。
2. **长期记忆的成败在检索**，不在存储。存进去容易，**取出来对不对**才是难点（第 14 章 RAG）。
3. **写入时机是设计决策**：每次会话都 `add_session_to_memory`？还是只存"有价值的"？存太多会把库污染，检索质量下降。
4. **记忆会过期/变错**：用户三个月前说喜欢靠窗，现在未必。长期记忆没有"过期"机制（要自己设计）。
5. **程序记忆自我改写有风险**：让 agent 改自己的指令（脚本 5 最后一格），可能越改越差——和第 4 章"循环交的是最后一版"是同一个病。

## 本机环境

| notebook | 内核 |
|---|---|
| 4 份 ADK | `Python (agentic-adk)` |
| 1 份 LangChain | `Python (agentic-book)` |

ADK 侧的 `VertexAiRagMemoryService` / `VertexAiSessionService` **需要 GCP 项目 + 认证**，
本机只做"能 import、缺配置时明确说明"的处理，不产生假输出。
