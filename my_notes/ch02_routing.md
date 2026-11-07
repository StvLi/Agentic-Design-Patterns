# 第 2 章 · Routing

对应 `chapter_notebooks/Chapter_02_Routing_(LangGraph).ipynb` 和
`Chapter_02_Routing_(Google_ADK).ipynb`。
这份笔记承接 notebook 里原来那一大段说明——notebook 只留主线，细读在这里。

## 一句话概括

先用**一次模型调用**把请求分类成 3 个标签之一，再用 **`RunnableBranch`**
按标签选一条分支去执行。

和第 1 章相比，变的是**控制流**：从「写死的顺序」变成「运行时按内容决定」。

## 文件名有坑

notebook 叫 `(LangGraph)`，但它**根本没用 LangGraph**，用的是 LangChain 的
`RunnableBranch`。书里真正的 LangGraph 条件边（`StateGraph` +
`add_conditional_edges`）并没给完整代码。

## 数据流全景（最重要的心智模型）

```
invoke({"request": "Book me a flight"})
   │
   ├── {"decision": router_chain,     ← 调 1 次模型，得 "booker"
   │    "request":  Passthrough()}    ← 原样保留输入，一个字节不改
   │        ↓ 合成一个 dict（并行算完再合并）
   │   {"decision": "booker",
   │    "request": {"request": "Book me a flight"}}
   │                    ↑ 这层嵌套就是 x['request']['request'] 的由来
   │
   ├── RunnableBranch 按 decision 选中 branches["booker"]
   │        ↓ 执行 booking_handler
   │   {"decision": ..., "request": ..., "output": "Booking Handler ..."}
   │
   └── lambda x: x['output']          ← 只抛结果字符串出去
```

## 本仓库相对书里原版的改动

| # | 改动 | 影响 |
|---|---|---|
| 1 | `info` 分类被**收窄**为「与订机票/酒店相关的信息问题」；原版是「所有其它信息问题」 | 与预订无关的普通知识问题，现在会落到 `unclear` |
| 2 | 演示用例 3 个 → 4 个（B 补了上下文，新增 D） | 覆盖了更多边界 |
| 3 | 修了 prompt 拼写 `destinatuion` → `destination` | 无语义变化 |

> ⚠️ 因为 prompt 已偏离原书，这个 notebook 的路由结果**不能**直接和
> `02_3_debug_lab.ipynb`（保留书里原版 prompt）对照。

## 执行时序：一次 `invoke()` 内部到底发生了什么

以 `"Book me a flight to London."` 为例：

| # | 环节 | 类型 | 花 token | 数据 |
|---|---|---|---|---|
| 1 | dict 展开 `{"decision":…, "request":…}` | 本地（构造并行） | ❌ | `{"request": "Book me…"}` |
| 2 | `Passthrough()` 分支 | 本地 | ❌ | 原样 `{"request": "Book me…"}` |
| 3 | 路由 prompt 插值 | 本地 | ❌ | → 2 条消息（system + user） |
| 4 | **`llm` 分类调用** | **网络** | ✅ | → `AIMessage("booker")` |
| 5 | `StrOutputParser` | 本地 | ❌ | → `"booker"` |
| 6 | dict 合并 | 本地 | ❌ | `{"decision": "booker", "request": {"request": "Book me…"}}` |
| 7 | `RunnableBranch` 谓词求值 | 本地 | ❌ | 选中 `branches["booker"]` |
| 8 | `branches["booker"]` | 本地 | ❌ | 调 handler → 追加 `output` 键 |
| 9 | `lambda x: x["output"]` | 本地 | ❌ | → 结果字符串 |

**关键数字**：整条路由链路**只花 1 次模型调用**。这比 Ch1 的 2 次还省——
因为路由只需要一个标签，不需要生成内容。

**关键风险**：第 4 步是唯一的模型调用，它错了，后面 5 步全都"正确地"执行错误决定。

## 实测记录（deepseek-flash, temperature=0, 收窄后的 prompt）

| 用例 | 请求 | 结果 | 是否与标签一致 |
|---|---|---|---|
| A booking | Book me a flight to London. | `booker`（稳定） | ✅ |
| B info | What is the capital of Italy? I'm planning a trip. | `info`（6/6） | ✅ |
| C unclear | Tell me about quantum physics. | `unclear`（稳定） | ✅ |
| D info | Tell me the earliest flight I can catch from New York to Roma next week. | `booker`/`info` **各占一半**（6 次） | ⚠️ 标签写 info |

三个值得停下来看的地方：

1. **用例 B 是靠补上下文才走进 info 的。** 同一句话去掉 "I'm planning a trip."
   之后，实测会落到 `unclear` —— 因为收窄后的 prompt 要求 info 必须与订机票/酒店
   相关。**一句话的上下文，决定了它走哪条分支。**
2. **用例 D 不稳定**：连跑 6 次，`booker` 3 次、`info` 3 次（`temperature=0`！）。
   这句话本身跨在两类边界上："查询最早能赶上的航班"算预订动作还是信息查询？
   prompt 没定义清楚，模型的推理路径每次也不完全相同，于是来回摇摆。
   这类不稳定最危险：**每次运行都"成功"，日志里找不到任何异常**。
3. **「标签」和「模型决定」是两回事。** 用例名里的标签是人写的期望；模型给的是
   它自己的判断。这段演示**没有任何断言**去比较两者 —— 这正是 Ch19 要解决的问题。

> 书里原版只有 3 个用例，且其中的 "C" 实际走的是 info（见 lab notebook 第 8 节）。
> 本 notebook 收窄 prompt 后，C 才真正落到 unclear。

## 八个关键点（书里没明说，但决定你怎么用它）

### ① 路由是「分类问题」，不是「生成问题」

模型在这里只输出一个标签，不生成内容。这意味着：可以用**小模型**甚至规则来做，
成本极低；可以单独评测（准确率、混淆矩阵），比评测生成质量容易得多；
可以用**非 LLM** 的方案替代（书里列的 Embedding / 规则 / ML 分类器）。

### ② 四种路由机制是**成本阶梯**，不是四选一

书里列了 LLM / Embedding / 规则 / ML 分类器。真实系统几乎都是**混合**的：

```
最常见格式  → 规则（~0 成本，覆盖 60-70%）
语义相近    → Embedding（本地向量比较）
长尾/模糊   → LLM 分类（1 次 API 调用）
```

先用免费的挡掉大多数，贵的那层只兜底。

### ③ `RunnablePassthrough()` 是「留一份原件」

分类和处理都要用同一段用户文本，所以必须原样透传。这个模式在 agent 里极常见
（Ch7 多 agent、Ch8 记忆都会再见到）。

### ④ 双重嵌套 `x['request']['request']` 的由来

不是笔误，是 `{"request": RunnablePassthrough()}` 把「原始输入 dict」整个透传
下来造成的。理解了这一点，LCEL 的数据形状就通了。

### ⑤ `.strip()` 是脆弱的防御

只能去空白。`'booker.'`、`'Booker'`、`"'booker'"` 全部会落到默认分支。
**正确做法是从严格到宽松的多级归一化**（精确 → 去包裹 → 文本内唯一命中 → 兜底），
见 `02_3_debug_lab.ipynb` 第 5 节。

### ⑥ 默认分支承载了两种不同语义

```python
branches["unclear"]   # 默认分支
```

它同时接收「模型**明确判定**为 unclear」（正常业务路径）和「模型输出了**意料之外**
的任何东西」（**异常信号，应该告警**）。两者合并成一个，就等于
**把异常伪装成正常业务**。生产环境必须拆开。

### ⑦ 路由错误是「静默降级」，比 Ch1 更隐蔽

Ch1 的错误至少可能表现为"格式不对"。路由错了，系统会**大方地返回一个看起来很
正常的回复**，只是答非所问。日志里一切正常。

> 这是 agent 可靠性最难的部分：**错误的执行路径不会自己喊疼。**

### ⑧ 类别边界上的样本会**摇摆**（实测）

见上面用例 D。这类问题的诊断难度远高于崩溃：不会报错、不会降级、每次输出都合理；
复现困难（再跑一次可能就"好了"）；只有**多次采样 + 统计**才能发现。

> 工程对策：对边界样本做**主动采样**（同一输入跑 N 次看一致性），发现摇摆就把
> 该类别拆细或补规则 —— 这正是 Ch19 评测要做的事。

## 书里没有的两个结构性缺陷

### 缺陷 A：单标签放不下复合意图

`"Book me a flight AND tell me about Italy"` 既是预订也是信息，但输出空间只有一个
标签。实测返回 `'booker'`，**信息那半被静默丢弃**。

这是**路由粒度设计问题**，不是模型能力问题。解法有三种：

1. 识别复合意图 → 拆成子任务（Ch6 Planning）
2. 多标签路由 → 输出列表 → 并行执行
3. 加一个 `multi` 类别 → 交给协调者

### 缺陷 B：没有回退路径

一旦分派出去，handler 无法说"我处理不了，退回协调者"。真实系统需要 ReAct 式的
**回退边**（Ch6、Ch12）。

## 想生产化还缺什么

| 缺什么 | 后果 | 正解在哪章 |
|---|---|---|
| 输出归一化 | 格式稍变即静默降级 | 本文第 ⑤ 点 / Ch18 |
| 意外输出的告警 | 异常伪装成正常业务 | Ch18 Guardrails |
| 复合请求分解 | 一半意图丢失 | Ch6 Planning |
| 路由正确性评测 | 不知道准确率多少 | Ch19 Evaluation |
| 回退机制 | 分类错了全错 | Ch12 Exception Handling |

## 本仓库里的文件

- `book_example.py` —— 与书一致的脚本
- `02_3_debug_lab.ipynb` —— 25 格，含 3 个实验（保留书里原版 prompt）
- `02_4_langchain_routing_lab.ipynb` —— LangChain 侧实验

---

# §2.4 · Google ADK（Auto-Flow 隐式路由）

对应 `chapter_notebooks/Chapter_02_Routing_(Google_ADK).ipynb`。
原代码在 ADK 2.13.0 上**跑不起来**，有三处必须改。逻辑未变。

## 它和 §2.3 的根本区别

| | §2.3 LangChain | §2.4 Google ADK |
|---|---|---|
| 路由逻辑在哪 | **你的代码里**（`RunnableBranch` 的 lambda） | **模型自己决定** |
| 代码里有 if/else | 有 | **一个都没有** |
| 模型做什么 | 只输出一个标签字符串 | 读 description，决定**转交给谁** |
| 触发条件 | 你写分支 | `sub_agents=[...]` 存在即自动开启 |
| 模型调用数 | 1 次 | **≥2 次**（协调者 1 次 + 子 agent 1 次）|

一句话：**§2.3 你当调度员；§2.4 你写招聘启事，让模型自己选人。**

代码里**没有任何路由判断**。你和模型之间的契约只有三样东西：
协调者的 `instruction`、每个子 agent 的 `description`、`sub_agents=[...]`。

## 本机适配清单（三处，缺一不可）

| # | 问题 | 原代码 | 改为 |
|---|---|---|---|
| 1 | 模型已下线 | `gemini-2.0-flash` | **`gemini-3.1-flash-lite`** |
| 2 | `create_session` 是协程 | `session_service.create_session(...)` | **`create_session_sync(...)`** |
| 3 | 免费层限流 5 RPM + 503 高负载 | 连续跑 4 个用例 | 用例间 **sleep** + 429/503 重试 |

> 第 2 条是原文**直接报错**的原因：不 await 就创建不了 session，之后
> `runner.run` 报 `Session not found`。

> 第 3 条的补充：**503 UNAVAILABLE 和 429 一样是瞬时的，都该重试**。
> 实测 `gemini-3.1-flash-lite` 上用例 A（预订）连撞两次 503，而同一次运行里
> 用例 B（信息）一次就过 —— 说明不是配额，是后端某个模型实例在抖动。
> 所以重试条件写的是 `429 / RESOURCE_EXHAUSTED / 503 / UNAVAILABLE`，
> 而不是只认 429。

## 可用模型实测（本机）

| 模型 | 结果 |
|---|---|
| `gemini-3.1-flash-lite` | ✅ 可用（本 notebook 默认） |
| `gemini-3.5-flash` | ✅ 可用 |
| `gemini-3-flash-preview` | ✅ 可用 |
| `gemini-2.0-flash`（书里用的） | ❌ 已下线 |
| `gemini-2.5-flash` / `-lite` | ❌ 对新用户关闭 |
| `gemini-3.8-flash` / `3.7-flash` | ⚠️ 503 高负载（可稍后重试） |

## Auto-Flow 内部到底发生了什么

以 `"Book me a hotel in Paris."` 为例，一次调用产生的事件序列：

| # | 事件 | 谁产生的 | 花 token |
|---|---|---|---|
| 1 | 用户消息进入 Coordinator | — | ❌ |
| 2 | **Coordinator 的 LLM 决策**：我应该转交给 Booker | Coordinator | ✅ **调用 1** |
| 3 | 转交事件（transfer to Booker） | 框架 | ❌ |
| 4 | **Booker 的 LLM 决策**：我应该调用 booking_tool | Booker | ✅ **调用 2** |
| 5 | 工具调用 `booking_handler("...")` | 框架 | ❌ |
| 6 | 工具返回结果回灌给 Booker | 框架 | ❌ |
| 7 | **Booker 组织最终回复** | Booker | ✅ **调用 3**（视实现） |
| 8 | `event.is_final_response()` 为真 | — | ❌ |

三个对比 §2.3 的关键数字：

1. **模型调用 2~3 次**，而 §2.3 只有 1 次。因为每个节点都是独立的 LLM。
2. **每次转交都会换 system instruction + tool set**，ADK 会警告请求前缀变了、
   缓存失效，整个 prompt 重发。所以 §2.4 的 token 成本明显更高。
3. **你无法在代码里"拦住"第 2 步** —— 没有可插入 if 的地方。想加显式规则，
   只能改 `instruction`/`description`（软约束）或换框架。

> 本机实测输出（`gemini-3.1-flash-lite`）：
> ```
>   >>> Booking Handler Called
> FINAL: I have initiated the booking process for a hotel in Paris.
>        Do you have any specific preferences for the hotel,
>        such as dates, budget, or preferred area?
> ```
> 它没有直接返回工具结果，而是**在工具结果基础上又生成了一句追问** ——
> 这就是第 7 步那次额外的模型调用。

## 四个实测发现（书里没有）

### ① 书里这版代码在 ADK 2.13 上**直接跑不起来**

```
RuntimeWarning: coroutine 'InMemorySessionService.create_session' was never awaited
→ An error occurred: Session not found: 614bfb9e-...
```

修法一行：`create_session` → `create_session_sync`。ADK 同时提供 `_sync` 变体的
还有 `get_session_sync` / `list_sessions_sync` / `delete_session_sync`，说明这不是
临时方案，而是官方给同步上下文的正式接口。

### ② 书里的 PDF 与仓库 notebook 不一致

| | PDF | 仓库 notebook |
|---|---|---|
| `run_coordinator` | `async def` + `await` | 普通 `def` |
| `main` | `async def`，`await main()` | 同步 |
| 依赖 | `nest_asyncio` | 无 |

**PDF 那版是对的思路但也不完整** —— 它 await 了 `create_session`，但 `runner.run`
仍是同步生成器（不能 `await`）。所以两版都需要你动手修。

### ③ `Content` 对象没有 `.text` 属性

书里那段看着啰嗦的兼容代码不是防御性编程，是必需的。本机实测直接访问
`ev.content.text` 会 `AttributeError: 'Content' object has no attribute 'text'`。

### ④ 免费层限流是按**模型**分开算的

```
429 RESOURCE_EXHAUSTED ... quotaId: GenerateRequestsPerMinutePerProjectPerModel-FreeTier
quotaValue: 5, model: gemini-3.6-flash
```

三个后果：

- **5 请求/分钟**，而 §2.4 每个请求要 2~3 次调用 → **实际只能跑约 2 个用例/分钟**
- quota 是 **PerProjectPerModel**，所以**换模型 = 换配额桶**（这也是默认选
  `flash-lite` 的原因，lite 类通常额度更高）
- 错误里带 `retryDelay`，可以据此自动退避（代码里已实现）

### ⑤ 无害的 stderr 噪音（别被吓到）

```
App "InMemoryRunner" can transfer between agents but has no context_cache_config...
Deprecated. Please migrate to the async method.
Task exception was never retrieved ... AsyncClient.aclose()
```

**它们都不影响结果。** 前两条是 ADK 自己的提示（缓存配置 + 同步接口弃用警告），
第三条是 ADK 与 httpx 在收尾阶段的已知问题（异步客户端没正常关闭）。

判断依据只有一条：**看 stdout 里有没有 `>>> Booking Handler Called` 和
`Coordinator Final Response:`**。有，就是成功。

## 一个书里自己承认的死代码

`unclear_handler` 在整段代码里**从未被注册成工具、也没被挂到任何 agent 上**。
书里第 11 页自己承认了："although the current coordinator logic doesn't explicitly
use it for delegation failure"。

这就是隐式路由的代价 —— §2.3 里它至少还是 `RunnableBranch` 的默认分支，
这里连"兜底"这个概念都消失了。模型转交失败会怎样，代码没有定义。

## 生产化缺口

| 缺什么 | 后果 | 正解 |
|---|---|---|
| **显式兜底路径** | 模型转交失败无人处理（`unclear_handler` 是死代码） | 自己加一层 wrapper（§2.3 风格） |
| **成本控制** | 每次转交重发全量 prompt | `context_cache_config`（ADK 自己的警告也这么说） |
| **可观测性** | 不知道模型为什么选了 Booker | tracing（Ch19） |
| **路由可测性** | 没法断言"该走 Booker" | 把每次 `transfer` 事件记录下来做评测 |
| **死循环保护** | agent 之间互相转交 | `max_depth` / 步数上限 |

## §2.3 和 §2.4 该选哪个

| 场景 | 选择 |
|---|---|
| 分支少、判据明确、要可测 | **§2.3 显式路由** |
| 能力多、语义模糊、要快速扩展 | **§2.4 Auto-Flow** |
| 安全关键（机器人动作、资金） | **§2.3**，且必须能写确定性校验 |
| 面向用户的开域问答分诊 | **§2.4** |

> 最实用的答案：**两者混用**。用 §2.3 的显式分支守住安全边界，
> 在"安全落点之内"用 §2.4 的 Auto-Flow 做能力选择。
