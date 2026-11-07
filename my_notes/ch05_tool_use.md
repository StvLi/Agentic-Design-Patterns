# 第 5 章 · Tool Use

本章有 **5 份 notebook**。它们不是"同一个例子换五个框架"，而是**工具的几种来源**各来一遍：

| 谁执行工具 | notebook |
|---|---|
| 你的进程（你写的函数） | `(LangChain)`、`(CrewAI)` |
| 厂商服务端（框架内建） | `(Google_Search)` |
| 厂商服务端（模型自己写代码） | `(Executing_Code)` |
| 外部服务（云端检索） | `(Vertex_AI_Search)` |

## 导读：按这个顺序读

**主线一句话：越往下读，你自己的代码越少。**

1. **`(LangChain)` —— 起点，可见性最高。** 工具是你写的函数，循环（问模型 → 执行 → 回灌）
   跑在你能看见的进程里。这份把"模型只**请求**、你才**执行**"讲透，后面几份都在剥这层可见性。
   重点：`@tool` 干的三件事——函数名→工具名、docstring→模型读的说明、类型注解→参数 schema。

2. **`(Google_Search)` —— 函数消失了。** `tools=[google_search]` 里放的不是函数，是框架
   导出的对象；执行发生在 Google 侧。于是"它到底调没调工具"**代码里看不出来**，
   只能查 `grounding_metadata`。这是本章第一个可观测性缺口。

3. **`(Executing_Code)` —— 连代码都是它写的。** 模型自己写 Python、服务端执行、拿回 stdout。
   API 形状也变了：它不进 `tools=[]`，是单独的 `code_executor=` 字段。
   有趣的是这份**反而看得见**——执行痕迹以结构化字段返回（`executable_code` /
   `code_execution_result`），比内建搜索更透明。

4. **`(Vertex_AI_Search)` —— 只剩一个 id。** 你的代码里只有一个 `data_store_id`，能力全在外部。
   而且书里的 `agents.VSearchAgent` 在 ADK 2.13 **已不存在**，现在它就是"又一个工具"，
   正好印证本章主线。（本机跑不了：需 Vertex AI 项目 + datastore。）

5. **`(CrewAI)` —— 换个框架看同一件事（可选，放最后）。** `@tool` 角色一样，变的是编排：
   `Agent` / `Task` / `Crew` 三层显式对象。需先装 crewai，所以排在最后。

> **一句话总结本章：工具越"内建"，你能观察和拦截的就越少。**
> 第 18 章 Guardrails 讲的就是怎么把这层可观测性加回来。
>
> notebook 里每份只留 4–6 处注释，全部围绕"这一章的增量"；完整展开在下面。

⚠️ 书里 `(LangChain)` 原 notebook 有**两段代码粘在同一个 cell 里**（第一段用 `gemini-pro`，
第二段用 `gemini-2.0-flash` 且带 `getpass`），是作者迭代留下的痕迹。本仓库只保留一份。

## 核心概念（一句话）

> **模型不会执行工具，它只会"请求"调用工具。**
> 真正去查数据库、调 API 的是**你的代码**。`AgentExecutor` 只是把这个循环包起来了。

## 工具调用的三步循环

```
① 把「问题 + 工具清单」发给模型
② 模型回复：最终答案，或 tool_calls（要调哪个工具、传什么参数）
③ 你执行工具 → 结果作为 ToolMessage 追加回对话 → 回到 ①
```

记住一句话就通了：**`AgentExecutor` = while 循环 + 工具注册表 + 迭代上限**。
后面每一个组件，都能对上这个循环的某个部件。

## 本机适配（两处，缺一不可）

| # | 问题 | 原代码 | 改为 |
|---|---|---|---|
| 1 | 模型 | `ChatGoogleGenerativeAI(model="gemini-pro")` | `common.make_llm()`（DeepSeek） |
| 2 | **API 搬家** | `from langchain.agents import create_tool_calling_agent, AgentExecutor` | **`from langchain_classic.agents import ...`** |

第 2 条在 LangChain 1.x 上是**硬报错**：

```
ImportError: cannot import name 'create_tool_calling_agent' from 'langchain.agents'
```

因为 v1 把 agent 重写了：`langchain.agents` 现在提供 `create_agent`，v0.3 的遗留 API
整体搬到了 `langchain_classic`。

## 实测：DeepSeek 支持工具调用

```python
llm.bind_tools([add]).invoke("What is 17 + 25?")
# → tool_calls=[{'name': 'add', 'args': {'a': 17, 'b': 25}}]   ✅
```

所以本章示例在本机**完全可跑**，不像第 3 章 ADK 那样受配额限制。

## 执行时序：一次 `ainvoke` 内部到底转了几圈

以 `"What is the capital of France?"` 为例（`verbose=True` 会把这条轨迹打出来）：

| # | 环节 | 花 token | 数据 |
|---|---|---|---|
| 1 | prompt 插值（system + human + scratchpad 空） | ❌ | 首次请求 |
| 2 | **模型决策 ①** | ✅ | 返回 `tool_calls=[{name:"search_information", args:{query:...}, id:"call_xxx"}]` |
| 3 | AgentExecutor 执行工具 | ❌ | 调 `search_information` → 本地字典查询 |
| 4 | 工具结果写入 scratchpad | ❌ | 作为 `ToolMessage(tool_call_id=...)` 追加 |
| 5 | **模型决策 ②** | ✅ | 这次没有 `tool_calls` → 给出最终答案 |
| 6 | 返回 `{"output": ...}` | ❌ | — |

**关键数字**：一次问答 = **2 次模型调用**（1 次决定调工具 + 1 次生成答案）。

工具本身不花 token，但**工具返回的内容会进入模型上下文**——所以返回大段数据会显著
增加成本。

## 为什么 `tool_call_id` 很重要

一次可能返回多个 `tool_calls`（实测 DeepSeek 支持）：

```
tool_calls = [
  {name: "clear_tool",  args: {...}, id: "call_A"},
  {name: "get_market_cap", args: {...}, id: "call_B"},
]
```

每个结果必须用**对应的 id** 回填：

```python
ToolMessage(content=result_A, tool_call_id="call_A")
ToolMessage(content=result_B, tool_call_id="call_B")
```

id 配对错了，模型会把 A 的结果当成 B 的答案——而且**不会报错**，只是答案莫名其妙。

## 七个关键点

### ① 模型只"请求"，不"执行"

模型输出的是 `tool_calls`——**一句结构化的话**，里面写着"我想调
search_information，参数是 {query: '...'}"。真正去执行的是 `AgentExecutor`
（或你手写的循环）。所以工具的**副作用完全由你控制**，这也是安全审查的入口（第 18 章）。

### ② docstring 决定"要不要调"，参数名决定"怎么调"

实测对照（同一个问题 "How is Apple's stock doing right now?"）：

| docstring | 结果 |
|---|---|
| docstring 只写 `Does stuff.` | **模型拒绝调用**，回："I don't have a way to look up live stock prices..." |
| docstring 写清用途与时机（`Look up the current stock price ... Use this whenever ...`） | ✅ 正确调用 |

**写不清楚 = 等于没有这个工具。** 这和你在第 2 章改 prompt 收窄 `info` 类别是同一件
事：**自然语言就是路由逻辑**。

### ③ 输入匹配策略必须容忍措辞差异

书里用 `simulated_results.get(query.lower())` 精确匹配。但实测模型传的是：

```
"What's the weather like in London?"      ← 完整问句，还多了个 "like"
```

精确匹配、甚至子串匹配（`"weather in london" in query`）**都会失手**。最后用的是
**词元全包含**：

```python
q_tokens = set(query.lower().split())
if set(key.split()) <= q_tokens: ...
```

> 教训：工具的入参格式，必须和模型**实际会传的东西**对得上。别假设模型会传你设计的
> 那个干净字符串。

### ④ 工具异常要变成数据，不能冒泡

实测（工具对未覆盖的城市抛 `ValueError`）：

```
step1 flaky_tool({'city': 'Tokyo'})        -> [ERROR] ValueError: not in coverage area
step2 flaky_tool({'city': 'Tokyo, Japan'}) -> [ERROR] ...
答案: I couldn't retrieve Tokyo's temperature — the service reports that
      Tokyo is outside its coverage area.
```

模型先**换了个参数重试**（`'Tokyo'` → `'Tokyo, Japan'`），再失败后**诚实说明原因**。

如果让异常直接冒泡，用户看到的就是 500 错误，而不是这段有用的解释。
**捕获 → 变成 `ToolMessage` 回灌**，是工具层的优雅降级。

### ⑤ 必须有迭代上限

模型可能反复调同一个失败的工具（实测确实重试了一次）。
`AgentExecutor(max_iterations=5)` 或手写循环的 `max_steps` 是必需的保险丝。

### ⑥ `agent_scratchpad` 不能省

它是中间过程的容器。少了它，模型看不到自己上一轮的动作，典型症状就是
**反复调同一个工具**直到撞上限。

### ⑦ 版本迁移是本书最大的实操障碍

本章 `ImportError` 只是第一处。LangChain 1.x 把 v0.3 的 agent 体系整体挪到了
`langchain_classic`。**读本书时遇到 `ImportError`，先怀疑 API 搬家，再怀疑版本。**

## 和前面几章的关系

| 章 | 模式 | 在本章的位置 |
|---|---|---|
| Ch1 | Prompt Chaining | 工具循环本身就是一条链（问→执行→再问） |
| Ch2 | Routing | docstring 决定模型"路由"到哪个工具 |
| Ch3 | Parallelization | 多个 `tool_calls` 可并发执行；书里三个查询用 `gather` 并发 |
| **Ch5** | **Tool Use** | 让 agent 从"只会说"变成"能动手" |

## 生产清单

| # | 事项 | 原因 |
|---|---|---|
| 1 | docstring 写"何时使用" | 决定模型调不调 |
| 2 | 参数加类型注解 | 否则 schema 生成不出 |
| 3 | 工具内校验参数 | 模型给的参数不可信 |
| 4 | 异常 → `ToolMessage` | 别让 agent 崩掉 |
| 5 | 设 `max_iterations` | 防死循环烧钱 |
| 6 | 返回值截断 | 返回值进上下文，越大越贵 |
| 7 | 记录每次 `tool_call` | 出事时唯一线索（第 19 章） |
| 8 | 工具数别太多 | 描述互相干扰，模型选错 |

## 一句话总结

> **Tool Use 的本质是"把自然语言变成结构化调用，再把结果变回自然语言"。**
> 模型负责两头（理解意图、组织答案），你的代码负责中间（执行）。
> 那条边界决定了安全、成本和可靠性——所以要清楚它画在哪。

## 本仓库里的文件

- `book_example.py` —— 与书一致的脚本
- `05_3_debug_lab.ipynb` —— 24 格：手写工具循环、description 如何决定调用、
  并行工具调用、工具报错自救

---

## 五种实现的状态（本仓库实测）

书里 Ch5 给了 5 份 notebook。它们的**价值不同**——不是"一个例子换五个框架"，
而是**工具的三种来源**各来了一遍：

| notebook | 工具来源 | 状态 |
|---|---|---|
| `(LangChain)` | 自己写的函数（`@tool`） | ✅ 已跑通 |
| `(CrewAI)` | 自己写的函数（CrewAI 的 `@tool`） | ⚙️ 环境已就绪（`Python (agentic-crewai)`），尚未执行 |
| `(Google_Search)` | **框架内建**（`google_search`） | ✅ 已跑通，带 grounding 证据 |
| `(Executing_Code)` | **模型自己写代码并执行** | ✅ 已跑通 |
| `(Vertex_AI_Search)` | 云端检索（`VertexAiSearchTool`） | ⛔ 本机不可跑（需 datastore） |

### 三种来源的区别（本章真正的主线）

1. **你写的函数**（LangChain / CrewAI）：模型输出 `tool_calls`，**你的进程**执行。
   好处是完全可控（能打日志、能校验参数、能限流），坏处是要自己维护 schema。
2. **框架内建工具**（`google_search`）：搜索在 **Google 服务端**跑，你的代码里看不到
   那一次调用。所以「模型有没有真的搜」要靠 `grounding_metadata` 判断——
   下面 Google_Search 那份就把它打出来了（本次 6 个搜索词、13 个引用来源）。
3. **代码执行**（`BuiltInCodeExecutor`）：模型自己写 Python，**服务端**执行后返回 stdout。
   注意它不在 `tools=[]` 里，而是单独的 `code_executor=` 字段。

> 一条贯穿的规律：**工具越"内建"，你能观察和拦截的就越少。** 第 18 章 Guardrails
> 讲的就是怎么把可观测性加回来。

### 实测输出摘要

**Google_Search**（`gemini-3.5-flash`，走中转站）：

```
[grounding] 搜索词: ['latest AI news 2026', 'artificial intelligence news November 2026']
[grounding] 引用来源: 13 个
```

这两个搜索词是**模型自己拟的**，不是我们给的提示——能直接看出它把一个问题拆成了几次检索。
（数字每次都不一样：跑过的几次分别是 2 词/13 源、6 词/13 源、2 词/7 源。所以**光看模型说了什么
不能证明它搜了**，要看 grounding。）

**Executing_Code**：两个用例都把三样东西分开打了：

```
--- Running Query: Calculate the value of (5 + 7) * 3 ---
  模型写的代码:  print((5 + 7) * 3)
  执行结果: Outcome.OUTCOME_OK -> 36
==> Final Agent Response: 36

--- Running Query: What is 10 factorial? ---
  模型写的代码:  import math / print(math.factorial(10))
  执行结果: Outcome.OUTCOME_OK -> 3628800
```

第二个用例选得好：`10!` 靠心算容易出错，它选择写代码而不是直接答，**正说明工具被真正启用了**。

### 书里这一章已经过时的两处

1. **`agents.VSearchAgent` 在 ADK 2.13 里不存在了**
   `from google.adk import agents` 里没有任何 VSearch 相关名字。现在的等价物是
   **`VertexAiSearchTool(data_store_id=...)` 挂到普通 `LlmAgent` 的 `tools=[]` 上**。
   也就是说「搜索 agent 是特殊类」的写法，已经统一回「搜索只是又一个工具」——
   正好印证本章主线。
   另外 `DiscoveryEngineSearchTool` 那条路还缺 `google.api_core`，装了才能 import。

2. **`gemini-2.0-flash` / `gemini-2.0-flash-exp` 均已下线**，全部换成了
   `common.adk_model()`（见 `.env` 的 `ADK_BACKEND`）。

### CrewAI 的专用环境（已备好）

本仓库现在有三个 venv，各管一摊：

| venv | 内核名 | 用途 |
|---|---|---|
| `~/agentic_dev_venv` | `Python (agentic-book)` | 主线：LangChain / LangGraph / DeepSeek |
| `~/agentic_adk_venv` | `Python (agentic-adk)` | Google ADK |
| `~/agentic_crewai_venv` | `Python (agentic-crewai)` | CrewAI（1.15.34，约 1.1 GB） |

**主环境没有被改动**。新 venv 也装了同一套 `.env` 启动引导
（`tools/install_book_bootstrap.sh ~/agentic_crewai_venv`），所以 `OPENAI_API_KEY` /
`OPENAI_API_BASE` 会自动指向 DeepSeek。

已验证跑通：CrewAI 1.15 + DeepSeek（模型名写 `openai/deepseek-flash`）。
运行时会看到一条 litellm 警告：

```
Using an approximate input token count for model 'deepseek-flash' ...
```

DeepSeek 不在 litellm 的内置 token 表里，所以它按"1 token ≈ 4 字符"估算——**无害**，
只影响它自己的成本统计。

### CrewAI 跑起来踩的两个坑

**① `RuntimeError: Agent execution was invoked synchronously from within a running event loop`**

VS Code 的 kernel 里本身有一个事件循环在跑，CrewAI 的同步 `kickoff()` 检测到就拒绝执行
（它直接抛错，而不是像 ADK 那样忍着）。它建议改用 `kickoff_async()`，但那要改写成协程。

修法：**放进独立线程**——线程里没有运行中的循环：

```python
def run_in_thread(fn):
    box = {}
    def _target():
        try: box["result"] = fn()
        except BaseException as e: box["error"] = e
    th = threading.Thread(target=_target); th.start(); th.join()
    if "error" in box: raise box["error"]
    return box["result"]

result = run_in_thread(financial_crew.kickoff)
```

**② `verbose=True` 在 CrewAI 1.15 里几乎不输出轨迹**

实测整个运行只剩一行 `[Finalize] todos_count=0`。想看"调了哪个工具、传了什么参数"，
得**订阅事件总线**（CrewAI 1.15 有 100 个事件类）：

```python
from crewai.events.event_bus import crewai_event_bus
from crewai.events.event_types import ToolUsageStartedEvent, ToolUsageFinishedEvent

@crewai_event_bus.on(ToolUsageStartedEvent)
def _on_tool_start(source, event):
    print(f"[事件] 工具调用: {event.tool_name} args={event.tool_args}")
```

订阅之后就拿到了和 LangChain 版同级的东西：

```
[事件] 工具调用: stock_price_lookup_tool args={'ticker': 'AAPL'}
[事件] 工具返回: 178.15
```

### 这一章的收尾：可观测性三档

这三份拼起来是一条完整的线：

| 框架 | 循环在哪 | 你能看到什么 |
|---|---|---|
| **LangChain** | 你写的 `AgentExecutor` | 日志天生可见：`Invoking: ... with {...}` |
| **CrewAI** | 框架的 `Crew` | 默认几乎看不见；**订阅事件才能拿回来** |
| **Google_Search / Vertex** | 厂商服务端 | **订阅也拿不到**——执行根本不在你的进程里 |

> 所以"可观测性"不是框架的固有属性，而是**执行位置决定的**。
> 你能订阅的前提是循环还跑在你这边。

### 补充：CrewAI 自带的 tracing ≠ 控制台输出

跑完会看到一块 "Tracing is disabled" 的面板，提示 `CREWAI_TRACING_ENABLED=true`。
**框架原生支持 tracing（OpenTelemetry），但它不是打到控制台**：

```python
# crewai_core/telemetry.py
CREWAI_TELEMETRY_BASE_URL = "https://telemetry.crewai.com:4319"

# crewai/telemetry/tracing/grants.py —— collector_url 由【服务端下发的 grant】决定
class TraceGrant:
    token: str
    collector_url: str
```

也就是说：开 tracing → 用账号换短期授权 → spans 上传到 CrewAI 云 → 在 `app.crewai.com` 看。

而且**不支持指向自己的 collector**：包里没有 `OTEL_EXPORTER_OTLP_ENDPOINT` 的处理，
`collector_url` 完全由服务端下发；`OTEL_SDK_DISABLED` 只用来关闭。

对学习场景，事件订阅仍然是对的选择（本地、即时、不上传）；tracing 的价值在团队场景
（跨运行持久化、可视化、可分享）。顺带留个心眼：**tracing 会把 prompt 和输出传到第三方云**，
可观测性和数据外流是一体两面 —— 第 18 章要划的就是这条线。

---

## 坑（都是实测踩出来的）

### ① 同一 notebook 里第二次执行格 → `RuntimeError: Event loop is closed`

症状：第一次跑全对，**再执行一次就全部（或随机几路）失败**，而且失败的那一路
不打任何提示，你只看到"少了一条答复"。

根因分两层，第二层才是关键：

1. 我们用 `asyncio.run` 跑协程 —— 它每次新建一个事件循环，跑完就关；
2. **`langchain_openai` 用 `@lru_cache` 缓存了一个模块级的 async httpx 客户端**，
   它绑定在**第一次使用它的事件循环**上。所以"每次新建 `llm`"没用 ——
   实例换了，缓存里的 httpx 客户端还是旧的。

修法：整个 kernel 会话**只用一个常驻事件循环**（后台线程 `run_forever`），
重复执行本格时复用它。见 `Chapter_05_Tool_Use_(LangChain).ipynb` 末尾：

```python
try:
    _LOOP, _THREAD                      # 重复执行本格时复用，不新建
except NameError:
    _LOOP = asyncio.new_event_loop()
    _THREAD = threading.Thread(target=_LOOP.run_forever, daemon=True)
    _THREAD.start()

def run_in_own_loop(async_fn):
    return asyncio.run_coroutine_threadsafe(async_fn(), _LOOP).result()
```

配套：`asyncio.gather(..., return_exceptions=True)` + 结尾汇总「N 个查询：成功 x，失败 y」，
把**静默丢失变成显式报告**。这比修好循环本身更重要 —— 不然你永远不知道有一路没了。

### ② `Deprecated. Please migrate to the async method.`

来源是 ADK 的 `InMemorySessionService`：它的**同步方法**（`create_session_sync` /
`get_session_sync` …）每调用一次就打一条 `logger.warning`。

notebook 是同步上下文，我们**有意**用 `_sync` 版，所以这条提示没有信息量。已在
`common/adk.py` 里给了精确静音（只调那一个 logger，不影响其它日志）：

```python
from common import quiet_adk_sync_warnings
quiet_adk_sync_warnings()          # 【适配】
```

### ③ 中转站偶发 `ReadTimeout` 与"空回合"

- **ReadTimeout**：网关抖动，不是你的代码问题。已用 `common.adk.py: retry_call()` 包住
  `runner.run`，瞬时错误自动退避重试。
- **空回合**：偶尔收到事件但**没有一个是 final response**，于是什么也不打印 ——
  又是一种"静默"。Google_Search 那份已加兜底提示：

```python
if answered == 0:
    print(f"⚠️ 收到 {len(events)} 个事件，但没有 final response。重跑这一格即可。")
```

> 这三条的共同点：**失败不吭声**。在 agent 系统里这比报错危险得多 ——
> 第 19 章（评测）和第 18 章（Guardrails）要解决的就是它。
