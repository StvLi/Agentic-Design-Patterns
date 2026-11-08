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
  （另有 `add_events_to_memory` / `add_memory` 两种更细的粒度，见下文）
- **读**：`search_memory(app_name, user_id, query)` —— **语义检索**（名字叫语义，
  但 `InMemoryMemoryService` 实际是词匹配，见文末小记）

## 三种长期记忆（对人的类比）

| 类型 | 记什么 | 典型实现 |
|---|---|---|
| **语义记忆** | 事实、偏好（"用户喜欢靠窗座位"） | 用户 profile（JSON）或事实文档集合 |
| **情节记忆** | 经历过什么、怎么做成的 | **few-shot 示例**：把过去成功的交互序列当例子 |
| **程序记忆** | 怎么做事的规则（系统提示本身） | **Reflection**：让 agent 读自己的指令 + 近期交互，自己改写指令 |

第三种最能体现"记忆真的改变行为"——脚本 5 的最后一格就是它。

## 长期记忆的写入：三个接口，颗粒度递进

| 接口 | 输入 | 写入范围 | 实现情况 |
|---|---|---|---|
| `add_session_to_memory(session)` | `Session` 对象 | 该会话**全部**有 content 的事件 | `@abstractmethod`，所有后端必须实现 |
| `add_events_to_memory(events=[...])` | `Event` 列表 | 你挑的子集，按**增量**（delta）处理 | base 默认 `NotImplementedError`；`InMemoryMemoryService` 实现 |
| `add_memory(memories=[MemoryEntry])` | `MemoryEntry` 列表 | 你自行构造的条目 | base 默认 `NotImplementedError`；**仅 `VertexAiMemoryBankService`** 实现 |

三者共用同一份输入（一次真实对话：用户说偏好，Talker 回一句），用同一查询
`'vegetarian'` 检索对照。

**① 整段会话**

```python
m1 = InMemoryMemoryService()
asyncio.run(m1.add_session_to_memory(session))
# 命中 2 条：author=user | "I am vegetarian and I never eat spicy food."
#            author=Talker | "I will make sure to suggest only mild, vegetarian options for you."
```

**② 只挑用户说的话**

```python
user_events = [e for e in session.events if e.author == "user"]
asyncio.run(m2.add_events_to_memory(app_name=APP, user_id=USER,
                                    events=user_events, session_id=session.id))
# 命中 1 条：author=user | "I am vegetarian and I never eat spicy food."
```

没有 Session 对象，`app_name` / `user_id` 必须自己传（① 是从 session 里读的）。
接口注释明确它是增量：`must not assume it represents the full session`。

**③ 直接写整理好的条目**

```python
asyncio.run(m3.add_memory(app_name=APP, user_id=USER, memories=[
    MemoryEntry(author="memory_writer", timestamp="2026-11-09T00:00:00Z",
                content=types.Content(role="user", parts=[types.Part(
                    text="Dietary preference: vegetarian, cannot eat spicy food.")])),
]))
# 命中 1 条：author=memory_writer | "Dietary preference: vegetarian, cannot eat spicy food."
```

在 `InMemoryMemoryService` 上直接调用会抛 `NotImplementedError`（不支持直接写记忆）。
上面是「把 `MemoryEntry` 转成 `Event` 再走 ②」的最小实现，只为看清语义。

**本质区别**：前两个接口的内容**必然来自真实对话**，只是筛选粒度不同；第三个的内容
**可以完全不来自对话**——可写入手工标注的偏好、业务规则、外部系统导入的画像，且
`author` / `timestamp` 全由调用方指定。

## 分桶：记忆与会话都按 `(app_name, user_id)` 隔离

会话存储是三层字典（`sessions/in_memory_session_service.py:84`）：

```python
self.sessions: dict[str, dict[str, dict[str, Session]]] = {}
#                app_name  user_id  session_id
```

记忆库同理，键是 `_user_key(app_name, user_id)`。`get_session_sync` 的三个参数正是
这棵树的三层路径。

**为什么不能用单一字段**——两个方向都会失效：

| 只保留 | 后果 |
|---|---|
| `user_id` | 同一用户在不同应用下的记忆混在一起，跨应用越权 |
| `app_name` | 该应用所有用户共享一个桶，检索到别人的偏好 |

`(app_name, user_id)` 是最小充分集：**租户 + 用户**。ADK 把两者都做成必填的
关键字参数（`search_memory(self, *, app_name, user_id, query)`），就是为了逼调用方
明确「查谁在哪个应用下的记忆」。

**`app_name` 的实际取值**：`APP = runner.app_name` 是一个字符串，默认
`'InMemoryRunner'`。它与 agent **无关**——agent 回答「谁来回答」，`app_name` 只标记
「数据属于哪个应用」。同一 `(app_name, user_id)` 下换多少个 agent，记忆仍是同一份；
同一个 agent 换个 `app_name`，就是一份全新的空记忆。

**不一致的代价**：建会话用 `"demo"`、检索传 `"memory_demo"`，数据进两个桶，
检索**返回 0 条且不报错**。

**另一个同类陷阱**：`InMemoryRunner` 内部自建一个 `InMemoryMemoryService()`
（`runners.py:2425`），构造函数不接受 `memory_service` 参数。所以我们自己 `new` 的
记忆库与 `runner.memory_service` 是**两个对象**，写进去的东西 agent 看不到。要共用
必须显式用 `Runner(app_name=..., agent=..., session_service=..., memory_service=...)`。

## 补：检索到的内容怎么进入 agent

脚本 3 里检索结果**没有**进入 agent，只被 `print`。真正进入的三条通路：

| 通路 | 谁触发 | 时机 |
|---|---|---|
| `preload_memory` | 框架，每次模型调用前自动 | 把结果拼成 `<PAST_CONVERSATIONS>` 段插入 `llm_request`，不进历史 |
| `load_memory` | 模型自己决定是否调用 | 结果作为 `function_response` 事件，下一次模型调用才在上下文里 |
| 自己的工具里调 `tool_context.search_memory` | 你的代码 | 同上 |

三者共同点：都发生在 **`LlmRequest` 组装之后、发给模型之前**。`preload_memory` 的具体
位置是 `LlmFlow.tool_request_processors` → `_agent_tools.request_processor` →
`tool.process_llm_request`。

⚠️ 在 `InMemoryMemoryService` 下 `preload_memory` 几乎无法分辨相关性：查询
`Where should I stay in New York?` 的分词含 `i`，而每条偏好里都有 `I`，于是 4 条全部
召回（上限 10 条）。记忆一多，每轮请求都会携带大量无关内容——管道不判断好坏，检索
质量完全取决于后端。

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

## 小记 · `InMemoryMemoryService` 的中文检索

该实现用 `re.findall(r'\w+')` 切词，而 Python 的 `\w` 包含中日韩字符。中文不以空格分词，
于是一整句被识别为**单一词元**，检索退化为「整句子串包含」判定：

| 查询 | 命中 |
|---|---|
| `'座位'` | 2 条（子串命中） |
| `'用户喜欢什么座位？'` | 0 条（整句不是任何记忆原文的子串） |

英文因天然以空格分词，词元可以部分重叠，同一份记忆换成英文问法即可命中：

| 查询 | 命中 |
|---|---|
| `'window seat'` | 2 条 |
| `'Which seat does the user prefer?'` | 2 条 |

但英文路径是词元**全等**匹配，不做词干还原：原文写 `hotels` 时查 `hotel` 命不中
（除非模型回覆里恰好用了单数形式）。所以单复数、时态变化都会影响结果。

故脚本 3 的示例改用英文文本。若必须用中文，可权宜地对查询插空格，替它完成分词：

```python
def spaced(query, n=2):
    q = query.strip("？?。.！!，,")
    return " ".join(q[i:i + n] for i in range(0, len(q), n))

# 同一句问话：原样 0 条，插空格后 2 条
for q in dict.fromkeys((query, spaced(query))):
    search_memory(app_name=..., user_id=..., query=q)
```

两处细节：

- `dict.fromkeys` 用于去重：两字查询插空格后与原串相同，不去重会重复打印。
- 按 2 字一格切分只是恰好成词；「图书馆」会被切成「图书 馆」，命中反而下降。

根治须换向量后端（`VertexAiRagMemoryService`），或引入 jieba 之类真正的分词器。
ADK 源码 `_extract_searchable_words` 的文档字符串已承认该限制，但只补了「中文中夹带
英文单词」一种情况（在拉丁与非拉丁文字边界处切分），纯中文未作处理。

## 本机环境

| notebook | 内核 |
|---|---|
| 4 份 ADK | `Python (agentic-adk)` |
| 1 份 LangChain | `Python (agentic-book)` |

ADK 侧的 `VertexAiRagMemoryService` / `VertexAiSessionService` **需要 GCP 项目 + 认证**，
本机只做"能 import、缺配置时明确说明"的处理，不产生假输出。
