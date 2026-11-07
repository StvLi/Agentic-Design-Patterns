# 第 3 章 · Parallelization

对应 `chapter_notebooks/Chapter_03_Parallelization_(LangChain).ipynb` 和
`Chapter_03_Parallelization_(Google_ADK).ipynb`。
这份笔记承接 notebook 里原来那一大段说明——notebook 只留主线，细读在这里。

## 和前两章的关系

| | Ch1 Chaining | Ch2 Routing | **Ch3 Parallelization** |
|---|---|---|---|
| 控制流 | 直线 | 分岔（选一条） | **同时走多条，再汇合** |
| 模型调用数 | 2 次串行 | 1 次分类 + 1 次执行 | **N 次并发 + 1 次汇合** |
| 前提 | 步骤有依赖 | 需要分类器 | **分支必须互不依赖** |
| 收益来源 | 可靠性 | 适应性 | **省时间** |

一句话：**前三章的"原子操作"到此凑齐了**——串行、分支、并发。
后面所有复杂 agent 都是这三种的组合。

## 形状必须记住：扇出 → 汇合

```
                    ┌── summarize ──┐
   topic ── fan-out ├── questions ──┤ fan-in ── synthesis ── answer
                    └── key_terms ──┘
```

- **扇出（fan-out）**：三条链同时启动，各自独立
- **汇合（fan-in）**：synthesis **必须等三路全部完成**——这一步注定串行

## 本机实测

| 方式 | 加速比 |
|---|---|
| 串行 3 次调用 | 1.00x（基准） |
| `RunnableParallel` | **2.8 ~ 2.9x** |

理论极限 3x。贴近极限说明瓶颈在**等网络**，不是算力。

## 执行时序：`invoke()` 内部到底发生了什么

| # | 环节 | 类型 | 花 token | 说明 |
|---|---|---|---|---|
| 1 | 输入 `{"topic": ...}` | — | ❌ | 一份输入，复制给三个分支 |
| 2 | `summarize_chain` | **网络** | ✅ | ┐ |
| 3 | `questions_chain` | **网络** | ✅ | ├ **三路并发** |
| 4 | `terms_chain` | **网络** | ✅ | ┘ |
| 5 | `RunnablePassthrough` | 本地 | ❌ | 原始 topic 原样带过 |
| 6 | 汇总成 dict | 本地 | ❌ | 4 个键：summary/questions/key_terms/topic |
| 7 | `synthesis_prompt` 插值 | 本地 | ❌ | 把 4 个值填进模板 |
| 8 | `llm` 汇合调用 | **网络** | ✅ | 串行，必须等前面全部完成 |
| 9 | `StrOutputParser` | 本地 | ❌ | → 最终字符串 |

**关键数字**：模型调用 4 次，其中前 3 次并发。总耗时 ≈ `max(3 条链) + synthesis`，
不是 `3 条相加 + synthesis`。本机实测：串行 6~10s → 并行 2.3~3.5s，**加速比 2.8~2.9x**。

**关键细节**：第 3 步的"并发"发生在**同一个线程**里。`RunnableParallel` 内部用线程池
（`invoke`）或事件循环（`ainvoke`）来切换任务，因为每条链绝大部分时间在**等网络响应**，
切换才有价值。

## 七个关键点

### ① 并行只对 I/O 密集有效

书里这段提醒非常重要（原文第 7 页）：

> "Note that asyncio provides **concurrency, not parallelism**. It achieves this on a
> single thread by using an event loop ... constrained by Python's Global Interpreter
> Lock (GIL)."

**并发（concurrency）≠ 并行（parallelism）。** 一个线程在多个任务之间切换，只是因为
它们在等待。如果三条链是纯 CPU 计算，GIL 会让它们排队，加速比接近 1.0，甚至更慢。

> 判断口诀：**收益来自"等待"，不是来自"计算"。**

### ② `RunnablePassthrough()` 不是可选项

`RunnableParallel` 的输出只含它算出来的键。原始输入默认被丢弃。下游若要 `{topic}`，
就必须显式透传——否则 `KeyError: 'topic'`。

### ③ `invoke` 和 `ainvoke` 都能并行

书里用 async 容易让人以为"异步才有并行"，其实不是。`invoke` 内部用线程池 executor，
一样并发。实测两者耗时量级相同。

**用 async 的真正理由**：不阻塞调用方。在 Web 服务里，阻塞式 `invoke` 会占住整个事件
循环，`ainvoke` 才能同时服务其他请求。

### ④ 并行的失败是 all-or-nothing

任一条分支抛异常，整条 `RunnableParallel` 就抛异常，**其他分支已经花掉的 token 白费**。

上生产时应该**在每个分支内部**加 `try/except`，让异常变成数据，而不是让整条链崩掉。

### ⑤ 瞬时并发 = 撞限流

并行把 3 次调用压成一个瞬间。下游若有 RPM 限制，串行"慢慢跑"没事，并行"一波打满"
直接 429。

> 本学习项目真实踩过：Google 免费层 5 RPM，ADK 一个请求就要 2~3 次调用，串行都不够用。
> **并行的第一成本不是 CPU，是配额。**

### ⑥ 汇合步骤注定串行

扇出再快，`synthesis` 也必须等三路齐全。所以 `总耗时 ≈ 最慢的那条分支 + 汇合生成`。
如果有一条分支特别慢，加速比会迅速趋近 1。**并行不等于均衡。**

### ⑦ 并行牺牲了"渐进式响应"

串行时用户能更早看到第一段结果；并行必须等最慢的那条才能继续。所以对"流式输出优先"
的场景，并行反而是倒退。

## 书里没给但很关键：LangGraph 的并行拓扑

书里说（§3.2）：

> "LangGraph ... allows you to define multiple nodes that can be executed from a single
> state transition, effectively enabling parallel branches."

这是对的，但**没给代码**（和 Ch2 的情况一样）。等价写法是：

```python
g.add_edge(START, "summarize")   # 三个节点都从 START 出发 = 扇出
g.add_edge(START, "questions")
g.add_edge(START, "key_terms")
for n in ("summarize", "questions", "key_terms"):
    g.add_edge(n, "synthesize")  # 三个都连汇合点 = 扇入（等全部完成）
```

完整可跑版本见 `03_3_debug_lab.ipynb` 第 8 节。

**和 `RunnableParallel` 的差别**：LangGraph 的并行由 **state** 驱动，每条分支写 state
的不同键，汇合节点天然能看到全部；好处是中间态显式可见、分支可单独重试、后续能接循环。

## 坑清单

| # | 坑 | 现象 | 处置 |
|---|---|---|---|
| 1 | 忘了 `RunnablePassthrough()` | `KeyError: 'topic'` | 显式透传 |
| 2 | 分支之间有隐藏依赖 | 结果错乱或报错 | 检查输入，有依赖就串行 |
| 3 | CPU 密集也上并行 | 没加速甚至更慢 | 记住 GIL |
| 4 | 一条分支抛异常 | 整体失败，token 白费 | 分支内 try/except 降级 |
| 5 | 瞬时并发撞 RPM | 429 | 算配额 / 加信号量限流 |
| 6 | 分支耗时悬殊 | 加速比趋近 1 | 拆分或异步化长任务 |
| 7 | notebook 里 `asyncio.run` | "cannot be called from a running event loop" | 线程内跑，或顶层 await |

## 一句话总结

> **并行不是"更快的串行"，而是"不同的拓扑"。**
> 先问两件事：这些任务之间有没有依赖？它们是不是主要在等 I/O？两个都是"是"，并行才有意义。

## 本仓库里的文件

- `book_example.py` —— 与书一致的脚本
- `03_3_debug_lab.ipynb` —— 25 格，含 6 个实验（含 LangGraph 并行拓扑实现）

---

# §3.3 下半 · Google ADK 版

对应 `chapter_notebooks/Chapter_03_Parallelization_(Google_ADK).ipynb`。
书里这份是**代码片段**（无 import、无 Runner、无 main），原样贴进来是一个
"贴进框架里才活"的骨架。

## 它在讲什么

```
SequentialAgent  (先并行，后汇总)
  ├── ParallelAgent            ← 三个研究员【同时】工作
  │     ├── 可再生能源研究员   (google_search)
  │     ├── 电动车研究员       (google_search)
  │     └── 碳捕获研究员       (google_search)
  └── SynthesisAgent           ← 读 state，合成一份带出处的报告
```

三个研究员之间没有任何数据依赖，所以能并行；汇总必须等它们全部完成，所以串行。

## 为什么这份不能直接跑

| 缺失 | 说明 |
|---|---|
| `import` | 片段里没有 import（原文交代了要放进 `agent.py`） |
| `Runner` / `Session` | ADK 把执行循环收走了，需要 `InMemoryRunner` 之类的宿主 |
| `GEMINI_MODEL` 的值 | 书里写 `gemini-2.0-flash`，该模型已下线 |

ADK 的设计取向是：**你声明结构，框架负责执行**。所以它的示例天然是"片段 + 宿主"的
形式，而不是自包含脚本。对照 §2.4 那份能跑的 ADK routing 示例——那份之所以能跑，
是因为我们补了 `InMemoryRunner` 和 session 创建。

> 本仓库的做法：结构声明部分补上 import 后**可以离线执行**（构造 agent 不花 token），
> 真正 `runner.run` 的那一格需要额度。

## 三个机制串起来看

ADK 的 agent 之间**不通过函数返回值传递数据**，而是通过 **session state**：

```
研究员 A ──output_key="renewable_energy_result"──┐
研究员 B ──output_key="ev_technology_result"─────┼──→ session state
研究员 C ──output_key="carbon_capture_result"────┘         │
                                                            ↓
汇总 agent 的 instruction 里写 {renewable_energy_result}
                ↓
        ADK 用 state 里的值替换占位符
```

**这个契约非常脆**：

- 键名写错 → 运行时占位符填不进去（不是编译错误）
- 三个研究员同时写 state → 如果键名撞了，会互相覆盖
- state 里有什么，取决于**执行顺序**——这正是为什么外层要包 `SequentialAgent`

这和 LangChain 版 `RunnableParallel` 的字典键名是同一类问题：
**靠字符串名字维系的数据契约，没有任何静态校验。**

## 和 LangChain 版的对照

| | LangChain（§3.3 上半） | ADK（§3.3 下半） |
|---|---|---|
| 并行的单位 | 一条链（一次 prompt + 一次调用） | **一个完整 agent**（多轮推理 + 工具） |
| 数据传递 | 内存里的 dict | **session state** |
| 契约形式 | `RunnableParallel` 的字典键名 | `output_key` ↔ `{占位符}` |
| 汇合方式 | `map_chain \| synthesis_prompt` | `SequentialAgent` 的第二段 |
| 能否独立跑 | ✅ 自包含 | ❌ 需要宿主 |
| 代码量 | 少 | 多（但结构更显式） |

**选型感觉**：任务简单、一次调用能完成 → LangChain 的 `RunnableParallel` 更轻；
需要多轮推理 + 工具、或者任务本身就是一个 agent → ADK 的粒度更合适。

## 现在能端到端跑通了（换中转站）

这一节长期卡在 Google 免费层的配额上。2026-11 接上 xiaojingai 中转站后跑通：
**三路研究 + 汇总，三份结果都进了 state**，0 报错。

切换方式只改 `.env` 一行（代码不动，见 `common/adk.py`）：

```
ADK_BACKEND=relay     # official = 官方 AI Studio（每模型 20 次/天）；relay = 中转站（付费额度）
```

## 三个实测发现

### ① 一个非确定性的 `KeyError`：契约缺 key 是硬崩，不是降级

第一次跑通了结构、却在汇合阶段炸掉：

```
KeyError: "Context variable not found: `ev_technology_result` in agent 'SynthesisAgent'."
```

但同样代码单独跑 `ParallelAgent` 是好的，重跑一次整条流水线也正常 ——
说明它是**竞态**：`ParallelAgent` 的某个子 agent 还没把 `output_key` 写进 state，
`SynthesisAgent` 已经开始解析 instruction 模板了。

**这是「契约靠字符串名字维系、没有静态校验」最硬的证据**：缺一个 key 不是降级，
而是整个流水线崩掉。而且它不可复现 —— 代码一个字没改，两次运行结果不同。

修法（notebook 里已实现）：建 session 时先给三个 key 占位。

```python
runner.session_service.create_session_sync(
    ..., state={k: "" for k in ("renewable_energy_result", "ev_technology_result", "carbon_capture_result")}
)
```

占位之后，缺的那一路会退化成一个**可见的警告**（notebook 会打印「这些 key 最后是空的」），
而不是一场崩溃。这和 Ch4 ADK 版遇到的「state 缺 key 时占位符填不进去」是同一类问题。

### ② `ParallelAgent` / `SequentialAgent` 已被标记弃用

在 ADK 2.13.0 上构造它们会打印：

```
DeprecationWarning: ParallelAgent is deprecated in favor of Workflow
                   and will be removed in a future version.
```

官方建议迁移到 `google.adk.workflow.Workflow`（graph-based workflow node）。
书里的写法**目前仍能工作**，但新项目建议直接用 `Workflow`。

### ③ 免费层几乎跑不动这一节（这也是当初卡住的原因）

三个研究员各自都要多轮（决定搜什么 → 搜索 → 总结），一次扇出实际是
6~9 次模型调用。而免费层的真实限制不是"每分钟"，是**每个模型每天 20 次**
（`GenerateRequestsPerDayPerProjectPerModel-FreeTier`）。

> 这也是第 2 章 §2.4 里撞 429 的同一个根因。
> **并行的第一成本不是 CPU，是配额** —— 这一节是这句话最好的例证。

## 如果想在本机跑它

```bash
# 1. ADK 装在独立的 ~/agentic_adk_venv 里
~/agentic_adk_venv/bin/pip list | grep google-adk

# 2. 把片段放进一个 agent 包
#    my_agent/agent.py   ← 这里放 root_agent
#    my_agent/__init__.py

# 3. 用 CLI 跑
adk run my_agent          # 或 adk web
```

⚠️ `google_search` 工具依赖 Google 的搜索接地（grounding），需要可用的 Gemini
API key；且免费层 5 RPM 撑不住 3 个多轮 agent 并发。
