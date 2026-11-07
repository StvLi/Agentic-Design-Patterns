# 《Agentic Design Patterns》学习路线图

> 仓库：https://github.com/evoiz/Agentic-Design-Patterns — Antonio Gulli 著（Springer, 2025）

## 材料就位情况

| 材料 | 位置 | 状态 |
|---|---|---|
| 全书 PDF（458 页，18.5 MB） | `Agentic-Design-Patterns/Agentic_Design_Patterns_Complete.pdf` | ✅ 仓库自带，无需爬取 |
| 纯文本全文（88 万字符） | `../text/book.txt` | ✅ 已用 pdftotext 提取 |
| 逐章拆分文本 | `../text/chapters/*.txt` | ✅ 29 个文件，方便按章精读 |
| 代码 notebook（58 个） | `chapter_notebooks/` | ⚠️ 51 个有真实代码，7 个是占位空壳 |

**占位空壳 notebook（无代码，仅附录）**：Appendix_A_Advanced_Prompting_Techniques.ipynb, Appendix_B_AI_Agentic_From_GUI_to_Real_world_environment.ipynb, Appendix_C_Quick_overview_of_Agentic_Frameworks.ipynb, Appendix_D_Building_an_Agent_with_AgentSpace.ipynb, Appendix_E_AI_Agents_on_the_CLI.ipynb, Appendix_F_Under_the_Hood_Reasoning_Engines.ipynb, Appendix_G_Coding_agents.ipynb

## 全书结构：21 章 + 7 附录

| 章 | 主题 | 词数 | 对应 notebook | 精读状态 |
|---|---|---|---|---|
| 0 | Introduction: What makes an AI system an Agent? | 6,216 | — | ✅ 已精读 |
| 1 | Prompt Chaining (Pipeline) | 3,867 | `Chapter_01_Prompt_Chaining_(Code_Example).ipynb`<br>`Chapter_01_Prompt_Chaining_(JSON_Example).ipynb` | ✅ 已精读（§1.3 代码 + §1.4 上下文工程） |
| 2 | Routing | 3,472 | `Chapter_02_Routing_(Google_ADK).ipynb`<br>`Chapter_02_Routing_(LangGraph).ipynb`<br>`Chapter_02_Routing_(Openrouter).ipynb` | ✅ 已精读（§2.3 显式路由 + §2.4 Auto-Flow） |
| 3 | Parallelization | 3,424 | `Chapter_03_Parallelization_(Google_ADK).ipynb`<br>`Chapter_03_Parallelization_(LangChain).ipynb` | ✅ 已精读（两版都跑通；ADK 段已解除配额阻塞） |
| 4 | Reflection | 3,304 | `Chapter_04_Reflection_(ADK).ipynb`<br>`Chapter_04_Reflection_(Iterative_Loop).ipynb`<br>`Chapter_04_Reflection_(LangChain).ipynb` | ✅ 已精读（三份 notebook 均已适配跑通） |
| 5 | Tool Use (Function Calling) | 5,069 | `Chapter_05_Tool_Use_(CrewAI).ipynb`<br>`Chapter_05_Tool_Use_(Executing_Code).ipynb`<br>`Chapter_05_Tool_Use_(Google_Search).ipynb`<br>`Chapter_05_Tool_Use_(LangChain).ipynb`<br>`Chapter_05_Tool_Use_(Vertex_AI_Search).ipynb` | 🔵 **复习中**（速通过一轮，现在回来扎实；5 份 notebook 已全部就绪） |
| 6 | Planning | 2,987 | `Chapter_06_Planning_(Code_Example).ipynb`<br>`Chapter_06_Planning_(Deep_Research_API).ipynb` | 🔵 进行中（CrewAI 版已跑通；Deep Research API 需 OpenAI 权限） |
| 7 | Multi-Agent Collaboration | 4,182 | `Chapter_07_Multi_Agent_(ADK_Gemini_AgentTool).ipynb`<br>`Chapter_07_Multi_Agent_(ADK_Gemini_Coordinator).ipynb`<br>`Chapter_07_Multi_Agent_(ADK_Gemini_Loop).ipynb`<br>`Chapter_07_Multi_Agent_(ADK_Gemini_Parallel).ipynb`<br>`Chapter_07_Multi_Agent_(ADK_Gemini_Sequential).ipynb`<br>`Chapter_07_Multi_Agent_(CrewAI_Gemini).ipynb` | 🔵 课件就绪（6 份全部跑通） |
| 8 | Memory Management | 5,069 | `Chapter_08_Memory_(ADK_Explicit_State_Update).ipynb`<br>`Chapter_08_Memory_(ADK_LlmAgent_output_key).ipynb`<br>`Chapter_08_Memory_(ADK_MemoryService_InMemory).ipynb`<br>`Chapter_08_Memory_(ADK_SessionService).ipynb`<br>`Chapter_08_Memory_(LangChain_LangGraph).ipynb` | ☐ |
| 9 | Learning and Adaptation | 3,131 | `Chapter_09_Adaptation_(OpenEvolve).ipynb` | ☐ |
| 10 | Model Context Protocol (MCP) | 4,315 | `Chapter_10_MCP_(ADK_FastMCP_Server).ipynb`<br>`Chapter_10_MCP_(FastMCP_Client_Agent_init).ipynb`<br>`Chapter_10_MCP_(FastMCP_Server_Example).ipynb`<br>`Chapter_10_MCP_(Filesystem_Example_agent).ipynb`<br>`Chapter_10_MCP_(Filesystem_Example_init).ipynb` | ☐ |
| 11 | Goal Setting and Monitoring | 2,881 | `Chapter_11_Goal_Setting_(Iteration).ipynb` | ☐ |
| 12 | Exception Handling & Recovery | 1,917 | `Chapter_12_Exception_Handling_(Fallback).ipynb` | ☐ |
| 13 | Human-in-the-Loop | 2,512 | `Chapter_13_Human_in_the_Loop_(Customer_Support).ipynb` | ☐ |
| 14 | Knowledge Retrieval (RAG) | 4,332 | `Chapter_14_Knowledge_Retrieval_(RAG_Google_Search).ipynb`<br>`Chapter_14_Knowledge_Retrieval_(RAG_LangChain).ipynb`<br>`Chapter_14_Knowledge_Retrieval_(RAG_VertexAI).ipynb` | ☐ |
| 15 | Inter-Agent Communication (A2A) | 3,048 | `Chapter_15_Inter_Agent_(A2A).ipynb`<br>`Chapter_15_Inter_Agent_(A2A_AgentCard_WeatherBot).ipynb`<br>`Chapter_15_Inter_Agent_(Sync_Streaming_Requests).ipynb` | ☐ |
| 16 | Resource-Aware Optimization | 3,474 | `Chapter_16_Resource_Optimization_(Code_Snippets).ipynb`<br>`Chapter_16_Resource_Optimization_(OI_Google_Search).ipynb` | ☐ |
| 17 | Reasoning Techniques | 6,226 | `Chapter_17_Reasoning_(CoT_Prompt).ipynb`<br>`Chapter_17_Reasoning_(Executing_Code).ipynb`<br>`Chapter_17_Reasoning_(Google_DeepSearch).ipynb`<br>`Chapter_17_Reasoning_(Self_Correction).ipynb` | ☐ |
| 18 | Guardrails / Safety | 5,154 | `Chapter_18_Guardrails_(ADK_Validate_Tool).ipynb`<br>`Chapter_18_Guardrails_(LLM_as_Guardrail).ipynb`<br>`Chapter_18_Guardrails_(Practical_Examples).ipynb` | ☐ |
| 19 | Evaluation and Monitoring | 4,647 | `Chapter_19_Evaluation_(Basic_Response_Evaluation).ipynb`<br>`Chapter_19_Evaluation_(LLM_as_Judge).ipynb` | ☐ |
| 20 | Prioritization | 2,296 | `Chapter_20_Prioritization_(SuperSimplePM).ipynb` | ☐ |
| 21 | Exploration and Discovery | 3,429 | `Chapter_21_Exploration_Discovery_(Agent_Laboratory).ipynb` | ☐ |

> **图例**：✅ 已精读并动手 ｜ 🔵 在读/复习中 ｜ ☐ 未开始　（⏸ 暂停 —— 当前无）
> 章节的「课件/实验产出」另见下文「各章产出」。

### 附录

| 附 | 主题 | 词数 | 代码状态 | 精读状态 |
|---|---|---|---|---|
| A | Advanced Prompting Techniques | 8,692 | ⚠️ notebook 为空壳，代码只在 PDF 里 | ☐ |
| B | From GUI to Real-World Environment | 2,223 | ⚠️ notebook 为空壳，代码只在 PDF 里 | ☐ |
| C | Quick Overview of Agentic Frameworks | 2,100 | ⚠️ notebook 为空壳，代码只在 PDF 里 | ☐ |
| D | Building an Agent with AgentSpace | 598 | ⚠️ notebook 为空壳，代码只在 PDF 里 | ☐ |
| E | AI Agents on the CLI | 1,556 | ⚠️ notebook 为空壳，代码只在 PDF 里 | ☐ |
| F | Under the Hood: Reasoning Engines | 3,974 | ⚠️ notebook 为空壳，代码只在 PDF 里 | ☐ |
| G | Coding Agents | 9,435 | ⚠️ notebook 为空壳，代码只在 PDF 里 | ☐ |

## 建议学习节奏（每章一个循环）

每章按 4 步走，产出物落在 `study/` 下：

1. **读**（我先把该章浓缩成 1 页：模式是什么 / 为什么 / 何时用 / 陷阱）
2. **辩**（我们就该模式在 Prime Lite 机器人场景下的适用性讨论——这是你们组的真实约束）
3. **写**（把书里示例重写成可跑的最小实现，优先不依赖厂商 SDK）
4. **测**（构造能暴露该模式失效边界的对照实验）

## 学习进度

| 章 | 主题 | 精读 | 课件/实验产出 | 结论（我记住了什么） |
|---|---|---|---|---|
| 0 | Introduction | ✅ | — | agent = 模型 + 工具 + 循环；四要素是后面所有章节的坐标系 |
| 1 | Prompt Chaining | ✅ | 脚本 + 9 格实验室 + 注释版 + README | 链只提升**可控性**，不提升正确性；错误静默传播 |
| 2 | Routing | ✅ | §2.3 / §2.4 注释版 + 3 个对照实验 + 24 格教学版 + README | 改一句自然语言，分类边界就平移，且没有 diff 可看；单标签放不下复合意图 |
| 3 | Parallelization | ✅ | 两版注释版 + 25 格实验室（6 个实验）+ README | 并行只对 I/O 有效；加速比上限受最慢分支限制；**契约缺 key 是硬崩不是降级** |
| 4 | Reflection | ✅ **已收束** | 导读 + 21 格双后端实验 + 3 份注释版 + 627 行实验记录 | 见下方「第 4 章收束结论」 |
| 5 | Tool Use | 🔵 **复习中** | 5 份注释版全部就绪（4 份可跑）+ 24 格实验室 + README | 工具三种来源；越内建越难观测 |
| 6 | Planning | 🔵 **进行中** | 导读 + 两份注释版（Code_Example 已跑通） | 书里的例子是**弱规划**：计划藏在 task 里，不是数据 |
| 7 | Multi-Agent | 🔵 **课件就绪** | 导读 + 6 份注释版（5 份 ADK + 1 份 CrewAI，全部跑通） | 三种通信方式：共享 state / 转交 / 当成工具调 |
| 8–21 | — | ☐ | 未开始 | — |

> 每章产出三类文件：**可跑脚本**（`chN_*/book_example.py`）、
> **调试实验室 notebook**（`0N_X_debug_lab.ipynb`）、
> 以及**书里原 notebook 的逐行注释版**（就地修改，`git checkout` 可还原）。

### 第 4 章收束结论（到此为止，不再深挖）

四组「生成者 × 批判者」对照，全部在本机实测：

| 组合 | 结果 | 耗时 |
|---|---|---|
| 弱生成 + 弱批判（本地 9B / 9B） | 第 3 轮批判段**退化成同一句重复**，被 2048 token 截断 | 数分钟，未收敛 |
| 弱生成 + 强批判（9B / DeepSeek，`format_duration`） | **9/14 → 13/14**，是真正的重构 | — |
| 强生成 + 强批判（DeepSeek / DeepSeek） | 一遍过，14/14；批判者说 `CODE_IS_PERFECT` | 数十秒 |
| 弱生成 + 强批判（9B / deepseek-v4-pro，`calculate_factorial`，10 轮） | **v1 满分 10/10，v2–v10 全崩，最终交付 0/10** | 792s |

`max_iterations` 提到 10 之后，弱+弱那组的批评建议确实"越来越多、越来越琐碎"，
弱批判+强生成则一遍过——这两条与上表一致。

**三条结论：**

1. **收敛取决于生成者的执行能力，不只是批判者的判断力。**
   第 4 行那次，强批判者每轮都诊断正确（第 2 轮就点出"那个巨大整数字面量让 docstring
   不可读且会被截断"，第 3–9 轮说"函数体缺失 / 三引号未闭合"，第 10 轮直接说"没提供代码"），
   但弱生成者每轮重蹈覆辙，**批评对它是"看得懂但做不到"**。
2. **循环交付的是"最后一版"，不是"最好的一版"。**
   机制上没有任何挑选或回滚，只保证"停在哪一版就交哪一版"。
   `max_iterations` 是预算保险丝，不是质量保险丝。
3. **多轮迭代与实时性天然冲突。**
   本地 9B 单次生成从 15.3s 涨到 73s（上下文里堆了历次代码和批评），
   10 轮里生成占 69% 的时间。**"弱模型 = 快"只在短上下文成立。**

**由此得到一个没做但记下来的改进方向**（留给以后，不阻塞进度）：

- **加确定性验证器**：每轮生成后先 `compile()` + 跑检查，失败就把**真实的 SyntaxError**
  喂回去，而不是把批判者的自然语言意见喂回去。这是修"看得懂但做不到"的正解。
- **改滑动窗口**：只带「上一版代码 + 最新批评」，不累积全部历史，把每轮延迟压平。

---

## 注释规范（增量式）

> 完整规范见 **`annotation_convention.md`**

**核心原则：只注释「本章首次出现的模式相关组件」。**

| 级别 | 标记 | 用法 |
|---|---|---|
| 新 | `★ 本章新增：...` | 展开讲（为什么这样设计、坑在哪） |
| 复用 | `（Ch<N>）...` | 一行交叉引用，不重复解释 |
| 无需 | 无注释 | 语法、显而易见的调用 |

配套的**已讲组件台账**在规范文档里（截至第 5 章），后续章节遇到这些组件只写交叉引用：
`ChatPromptTemplate`、LCEL 的 `|`、`StrOutputParser`（Ch1）；
`RunnableBranch`、`RunnablePassthrough(.assign)`、ADK `output_key`/Auto-Flow（Ch2）；
`RunnableParallel`、并发≠并行、ADK `ParallelAgent`/`SequentialAgent`（Ch3）；
`@tool`、`bind_tools`/`tool_calls`（Ch5）。

### notebook 形态：一格简报 + 代码

> 立这个规矩的原因：早期版本把大量讲解塞进 markdown 格和代码横幅注释里，
> 结果是「看原版很快懂用意，看改版经常 get lost」。现在反过来：

| 放哪 | 放什么 |
|---|---|
| notebook 的 **1 格 markdown** | 3–5 行：这个程序干什么、kernel 选哪个、【适配】了哪些、细读去哪看 |
| **代码格** | 书里代码骨架原样保留，只留 `★ 本章新增` / `（ChN）` / `【适配】` 三种短注 |
| **`chN_*/README.md`** | 全部长文：执行时序表、关键点、实测记录、坑清单、生产化缺口 |

已按此重做 **9 份 notebook**（Ch1 / Ch2×2 / Ch3×2 / Ch4×3 / Ch5）：
markdown 总量从约 **2.1 万字压到 2,541 字**，长文落在 4 份 README（约 40 KB），
全部 0 错误执行、输出已保存。

## 各章产出

### 第 1 章 Prompt Chaining（✅ 完成）

| 文件 | 用途 |
|---|---|
| `../../ch01_prompt_chaining/book_example.py` | 书里 §1.3 的可跑版本 |
| `../../ch01_prompt_chaining/01_3_debug_lab.ipynb` | 9 格拆解版，含 LCEL 手工展开 |
| `ch01_prompt_chaining.md` | 本章细读笔记（执行时序表 / 六个关键点 / 生产化缺口） |
| `Agentic-Design-Patterns/chapter_notebooks/Chapter_01_Prompt_Chaining_(Code_Example).ipynb` | 原书逐行注释版 |

### 第 2 章 §2.4 Google ADK（需 Google 凭证）

| 项目 | 状态 |
|---|---|
| 独立 venv `~/agentic_adk_venv` | ✅ google-adk 2.13.0 / google-genai 2.33.0 |
| 书里 §2.4 全部 import | ✅ 6/6 解析通过（API 名未变） |
| kernel `Python (agentic-adk)` | ✅ 已注册 |
| 到 Google API 网络连通 | ✅ 已通（403 = 缺凭证，非网络问题） |
| 凭证 | ✅ 已配置并验证（`../../docs/google_credentials.md`） |
| **跑通状态** | ✅ **端到端跑通**（booking → Booker，info → Info） |

**§2.4 必须做的三处适配**（原代码在 adk 2.13.0 上直接报错）：

| # | 问题 | 改为 |
|---|---|---|
| 1 | `gemini-2.0-flash` 已下线 | `gemini-3.1-flash-lite`（或 `gemini-3.5-flash`）|
| 2 | `create_session` 是协程，不 await → `Session not found` | `create_session_sync(...)` |
| 3 | 免费层 **5 请求/分钟**（按模型分桶） | 用例间 sleep + 429 自动退避 |

**可用模型实测**（2026-10-29）：

| 模型 | 状态 |
|---|---|
| `gemini-3.1-flash-lite` / `gemini-3.5-flash` / `gemini-3-flash-preview` | ✅ 可用 |
| `gemini-2.0-flash`（书里用的） | ❌ 已下线 |
| `gemini-2.5-flash` / `-lite` | ❌ 对新用户关闭 |
| `gemini-3.8-flash` / `3.7-flash` | ⚠️ 503 高负载 |

> 实测输出：
> - `"Book me a hotel in Paris."` → `>>> Booking Handler Called` → 回复已处理
> - `"What is the highest mountain..."` → `>>> Info Handler Called` → Mount Everest
>
> 说明 Auto-Flow 的**模型驱动委派**确实生效。

> 单独建 venv 的原因：`google-adk` 要求 `websockets>=15.0.1,<16`，
> 会把 `agentic_dev_venv` 里的 17.3 强制降级。

### 第 4 章 Reflection（✅ 已收束，2026-11）

| 文件 | 用途 |
|---|---|
| `ch04_reflection.md` | **章节导读**（11 节，含实测数据与踩坑记录） |
| `../../ch04_reflection/book_example.py` | 书里 §4.3 单轮反思链（可跑） |
| `../../ch04_reflection/04_3_debug_lab.ipynb` | **21 格 / 7 节**，双后端实验 |
| `../../ch04_reflection/local_llm.py` | 本地 Qwen3.5-0.8B 接入（Ollama 原生 API） |

**为什么需要双后端**：强模型做反思实验，基线一次就满分，反思没有改进空间。

| 后端 | 基线（`format_duration`，14 项） | 反思后 |
|---|---|---|
| DeepSeek-V4.1-Flash | 14/14 | 14/14（批判者说 PERFECT） |
| Qwen3.5-0.8B 本地 | **8/14** | 8/14（批判者**也**说 PERFECT） |

**本章最重要的测量 —— 批判者水平决定一切**（人为注入 3 个已知缺陷）：

| 批判者 | 命中率 |
|---|---|
| Qwen3.5-0.8B | **0/3**（三个都说"完美"） |
| DeepSeek-V4.1-Flash | **3/3** |

→ 结论：**生产者可以弱，批判者必须强**（非对称架构），且**有客观测试时优先用测试当裁判**。

**本地模型环境**（用户级安装，不碰系统/主环境）：`~/.local/ollama/` + GGUF
`~/.local/models/Qwen3.5-0.8B-Q4_K_M.gguf`，模型 `qwen3.5-0.8b:latest`，实测 100% GPU。

**书里三份 notebook 已全部适配为可运行 + 中文注释**（2026-10-31）：

| notebook | 内核 | 关键适配 | 状态 |
|---|---|---|---|
| `(LangChain)` | agentic-book | `ChatOpenAI`→`make_llm`；`sys.exit`→`raise`；`asyncio.run`→线程内跑 | ✅ 跑通 |
| `(Iterative_Loop)` | agentic-book | 去掉 `OPENAI_API_KEY` 检查；停止条件改精确匹配；默认本地 9B | ✅ 跑通（3 轮 / 6 次调用 / 213s）|
| `(Google_ADK)` | **agentic-adk** | **补 Runner**；`{draft_text}` 占位符；收全部 final response | ✅ 跑通（2 次调用）|

**适配中发现的原书问题**：

| # | 问题 | 证据 |
|---|---|---|
| 1 | ADK 版 reviewer **看不到草稿** | 原书只写"读取 state 键 draft_text"，没有 `{draft_text}` 占位符。ADK 源码 `inject_session_state` 只替换 `{var}` 形式的变量 → 草稿从未进入 prompt |
| 2 | ADK 版**没有 Runner** | 片段只定义 agent，注释说"generator runs..."但无任何驱动代码 |
| 3 | `is_final_response()` 每个子 agent 都为真 | `SequentialAgent` 里生成者和批判者各有 final response；break 在第一个会拿到**草稿**而非评审结论 |
| 4 | `asyncio.run` 在 notebook 里报错 | 事件循环已在运行 |

**环境补充**：ADK venv 装了 `httpx[socks]`（启动引导会把 `socks://` 改写为 `socks5://`，缺 socksio 会报错）。

### 第 5 章 Tool Use（🔵 **复习中**，课件全就绪）

> 因短期项目需要优先整理。**DeepSeek 支持工具调用**（实测 `tool_calls` 正常）。

| 文件 | 用途 |
|---|---|
| `../../ch05_tool_use/book_example.py` | 书里 §5.3 的可跑版本（LangChain + DeepSeek） |
| `../../ch05_tool_use/05_3_debug_lab.ipynb` | **29 格 / 8 节**，含 5 组实测；第 8 节是带参数约束的时区工具练习 |
| `.../Chapter_05_Tool_Use_(LangChain).ipynb` | 原书逐行注释版（已适配） |
| `ch05_tool_use.md` | 本章细读笔记（执行时序 / 七个关键点 / 五种实现状态表 / 与前四章的关系） |

**书里五份 notebook 现在全部就绪**（1 格简报 + 1 格代码，中文注释克制，只标 ★本章新增 / 【适配】）：

| notebook | 工具来源 | 状态 |
|---|---|---|
| `(LangChain)` | 自己写的函数 `@tool` | ✅ 跑通 |
| `(CrewAI)` | 自己写的函数（CrewAI 版 `@tool`） | ⚠️ 代码就绪，**未执行**（需 `pip install crewai`） |
| `(Google_Search)` | 框架内建 `google_search` | ✅ 跑通，带 grounding 证据（本次 2 个搜索词 / 7 个来源） |
| `(Executing_Code)` | 模型自己写代码并在服务端执行 | ✅ 跑通（`(5+7)*3 → 36`、`math.factorial(10) → 3628800`） |
| `(Vertex_AI_Search)` | 云端检索 | ⛔ 本机不可跑（需 Vertex AI 项目 + datastore） |

**这一章真正的主线**：工具的**三种来源**——自己写的函数、框架内建、模型自己写代码执行。
规律是**工具越"内建"，你能观察和拦截的就越少**（内建工具调没调，代码里看不到，
只能看 grounding），第 18 章 Guardrails 就是把这层可观测性加回来。

**两处书里已过时的 API**：

1. `agents.VSearchAgent` 在 ADK 2.13 里**不存在**（`google.adk.agents` 里没有任何 VSearch 名字）。
   现在的等价物是 `VertexAiSearchTool(data_store_id=...)` 挂到普通 `LlmAgent` 的 `tools=[]` 上——
   正好印证"搜索只是又一个工具"。
2. `gemini-2.0-flash` / `gemini-2.0-flash-exp` 均已下线 → 统一走 `common.adk_model()`。

**lab 的核心是"手写工具循环"**（20 行，不用 agent 框架）：
`bind_tools` → `tool_calls` → 手动执行 → `ToolMessage(tool_call_id=...)` → 再问。
看清 `AgentExecutor` 到底包了什么。

**必须知道的版本坑**：

```python
from langchain.agents import create_tool_calling_agent, AgentExecutor   # ❌ ImportError
from langchain_classic.agents import create_tool_calling_agent, AgentExecutor  # ✅
from langchain.agents import create_agent                               # ✅ v1 新 API
```

**四个实测发现**：

| # | 发现 | 证据 |
|---|---|---|
| 1 | **docstring 决定模型调不调** | 含糊 `"Does stuff."` → 模型拒绝调用；写清用途 → 正确调用 |
| 2 | 模型会传**完整问句**，不是干净 key | `"What's the weather like in London?"` 使精确/子串匹配失效 → 改用词汇全包含 |
| 3 | **工具异常变数据后模型会自救** | 传 `'Tokyo'` 失败 → 自动换 `'Tokyo, Japan'` 重试 → 再失败后诚实说明"不在覆盖范围" |
| 4 | 一次可返回**多个 `tool_calls`** | 实测同一轮返回 2 个（股价 + 市值） |

**三种写法对比**（lab 第 6、7 节）：手写循环 / `langchain_classic.AgentExecutor` / `create_agent`。

### 第 3 章 Parallelization（🟡 课件就绪）

| 文件 | 用途 |
|---|---|
| `../../ch03_parallelization/book_example.py` | 书里 §3.3 的可跑版本（LangChain，DeepSeek） |
| `../../ch03_parallelization/03_3_debug_lab.ipynb` | **25 格，6 个实验**，含实测加速比 |
| `.../Chapter_03_Parallelization_(LangChain).ipynb` | 原书逐行注释版（已适配） |
| `.../Chapter_03_Parallelization_(Google_ADK).ipynb` | ✅ **已端到端跑通**（走中转站）：三路研究 + 汇总，三份结果都进了 state |
| `ch03_parallelization.md` | 本章细读笔记（七关键点 / 坑清单 / 两版对照） |

**实测数据**：串行 6~10s → 并行 2.3~3.5s，**加速比 2.8~2.9x**（理论上限 3x）。

**lab 的 6 个实验**：

1. 串行 vs 并行计时（真实加速比）
2. `invoke` vs `ainvoke` —— 都能并行，量级相同
3. `RunnablePassthrough()` 缺失会怎样（零成本验证 `KeyError`）
4. **一条分支失败 → 整体失败（all-or-nothing）**，其他分支 token 白费
5. 并行的代价：瞬时并发撞限流（引第 2 章真实 429 经历）
6. LangGraph 三路扇出 + 计时，与 `RunnableParallel` 对照

**两个实测发现**：

| 发现 | 说明 |
|---|---|
| `ParallelAgent` / `SequentialAgent` 在 ADK 2.13 **已弃用** | 官方建议迁移到 `Workflow`（graph-based workflow node），书里写法仍能工作但会告警 |
| ADK 版是**片段**不是脚本 | 无 import / 无 Runner，原文交代要放进 `agent.py` 并用 `adk web` 启动 |

### ✅ 第 3 章 ADK 部分：已解除阻塞（2026-11，中转站）

**原因**：现有 Gemini API 的配额撑不住这一节的调用量。

算一下就知道为什么：

| 项 | 数量 |
|---|---|
| 并行研究员 | 3 个 |
| 每个研究员最少轮次 | 2（决定搜什么 → 汇总）|
| 每个研究员带 `google_search` | 工具调用还会额外消耗 |
| 合成 agent | 1~2 次 |
| **合计** | **7~11 次模型调用** |
| Google 免费层上限 | **5 请求/分钟**（按模型分桶）|

也就是说：**一次扇出就会超限**，而并行恰恰把调用压缩到同一个瞬间，
比串行更容易撞 429。

**已完成的**：注释解读（三个机制：`output_key` → session state → `{占位符}`）、
与 LangChain 版的差异对照、API 可用性验证（不调模型）、弃用警告实测。

**未完成的**：真实端到端运行。

> 2026-11 复核：notebook 重做后补了 import，第一格（构造 agent）**已能离线跑通、0 token**；
> 第二格真正 `runner.run` 仍返回 `_ResourceExhaustedError`（429），异常已捕获并打印原因，
> 不会中断 notebook。也就是说「结构对不对」现在可以验证了，「结果好不好」还得等额度。

**解除阻塞的条件（任选其一）**：

1. 升级 Gemini API 到付费层（配额足够）
2. 把 `google_search` 换成不依赖接地的本地工具，并减少研究员数量
3. 用**不调模型**的 mock agent 验证 `ParallelAgent` 的调度语义（零配额）

### 第 2 章 Routing（🟡 课件就绪）

| 文件 | 用途 |
|---|---|
| `../../ch02_routing/book_example.py` | 书里 §2.3 的可跑版本（RunnableBranch 版） |
| `../../ch02_routing/02_3_debug_lab.ipynb` | 25 格，含 3 个对照实验 |
| `../../ch02_routing/02_4_langchain_routing_lab.ipynb` | **24 格教学版**：L0 纯 Python → L1 `RunnableBranch` → L2 `RunnableLambda` 分派 → L3 JSON+校验 → 附录 LangGraph 对照 |
| `Agentic-Design-Patterns/chapter_notebooks/Chapter_02_Routing_(LangGraph).ipynb` | 原书逐行注释版 |
| `ch02_routing.md` | 本章细读笔记（§2.3 数据流全景 / 八个关键点 / §2.4 Auto-Flow / 两版选型） |

**第 2 章的适配改动**（均以 `【适配】` 标注在代码里）：

1. `ChatGoogleGenerativeAI(model="gemini-2.5-flash")` → `make_llm()`（DeepSeek）
2. `sys.path` 改为**向上搜索** `../../common/` 目录，不再依赖 kernel cwd
3. 保留书里 `.strip()` 逻辑不动，但在注释里标明其脆弱性，
   并把鲁棒版实现放进 lab notebook 做对照

**你对第 2 章 prompt 的改动（已同步到注释）**：把 `info` 类别从
「所有其它信息问题」**收窄**为「与订机票/酒店相关的信息问题」。

| 输入 | 原版 prompt | 收窄后 |
|---|---|---|
| `What is the capital of Italy?`（无上下文） | `info` | **`unclear`** |
| `...I'm planning a trip.`（补上下文） | — | `info` |
| `Tell me about quantum physics.` | `info` | **`unclear`**（终于对上书里的标签） |
| `Tell me the earliest flight I can catch...` | — | **不稳定：booker/info 各半（6 次）** |

> 教训：改一句自然语言，整个分类边界就平移了，**没有任何测试会因此失败**。
> 这就是 prompt 工程最危险的地方 —— 行为变更没有 diff 可看。

**第 2 章实测发现的三个问题**（书里代码看不出来）：

| # | 问题 | 证据 |
|---|---|---|
| 1 | 书里用例 C **没有测到 unclear 分支** | `"Tell me about quantum physics."` → `info`（符合 prompt 定义），`unclear_handler` 全程未执行 |
| 2 | `.strip()` 判据过窄 | `'booker.'` / `'Booker'` / `"'booker'"` 全部静默落到默认分支 |
| 3 | 单标签放不下复合意图 | `"Book me a flight AND tell me about Italy"` → 只返回 `'booker'`，信息意图被丢弃 |

## 分组建议

- **第 1 组 基础积木（1-4）**：Chaining / Routing / Parallelization / Reflection —— 这 4 个是所有编排的原子操作，先吃透，后面 17 章都在复用它们。
- **第 2 组 与外部世界接口（5, 10, 14）**：Tool Use / MCP / RAG —— 决定 agent 能"够到"什么。
- **第 3 组 单个 agent 的智能（6, 8, 9, 11, 17）**：Planning / Memory / Adaptation / Goal / Reasoning —— 一个 agent 内部如何思考与记忆。
- **第 4 组 多 agent 与协议（7, 15）**：Multi-Agent / A2A —— 拓扑与通信。
- **第 5 组 生产化（12, 13, 16, 18, 19）**：异常 / HITL / 资源 / 护栏 / 评测 —— 上生产必备，你们最该重视的一组。
- **第 6 组 前沿（20, 21）**：Prioritization / Exploration。

## 与本组的直接相关性（我的判断）

作者在导言里给出 5 个未来假设，其中 **假设 3「Embodiment 与物理世界交互」** 就是 Prime Lite 赛道：

> "agents breaking free from their purely digital confines to operate in the physical world... use vision sensors to perceive the problem, access a library of plumbing knowledge to formulate a plan, then control its robotic manipulators with precision"

另外 **附录 B「AI Agentic: From GUI to Real world environment」** 是全书唯一直接讲具身/真实环境的附录，建议和正文最后一起读。

## 环境现状（已验证 ✅）

虚拟环境：`~/agentic_dev_venv`  ·  Python 3.12.3  ·  与系统 site-packages 隔离

| 组件 | 版本 | 状态 |
|---|---|---|
| langchain / langchain-core | 1.4.7 / 1.7.0 | ✅ |
| langchain-openai | 1.7.2 | ✅ |
| langchain-community / -classic | 0.4.2 / 1.0.8 | ✅ |
| langgraph | 1.2.17 | ✅ |
| openai SDK | 3.37.0 | ✅ |
| faiss-cpu（Ch14 RAG） | 1.15.1 | ✅ |
| fastmcp / mcp（Ch10） | 4.1.4 / 2.3.0 | ✅ |
| jupyterlab / ipykernel | 4.6.4 / 7.4.0 | ✅ |
| numpy / pydantic | 2.5.4 / 2.14.1 | ✅ |

**Jupyter kernel**：已注册 `agentic-book`（显示名 `Python (agentic-book)`），notebook 里选它即可。

### VS Code 已配置（Notebook 调试可用）

```bash
./run_in_vscode.sh          # 打开工作区
```

| 配置 | 内容 |
|---|---|
| `.vscode/settings.json` | 解释器指向 venv；`jupyter.notebookFileRoot` = 仓库根（这样 `import common` 可用）；自动加载 `.env` |
| `.vscode/launch.json` | 3 个调试配置，其中 `justMyCode: false` 可单步进入 langchain / openai 内部 |
| `../../ch01_prompt_chaining/01_3_debug_lab.ipynb` | §1.3 的可调试拆解版，9 个代码格，已跑通 |

**调试 notebook 的操作**：点行号左侧打红点 → `Ctrl+Alt+Enter`（或 Runtime ▸ Debug Cell）。
注意要用 **kernel `agentic-book`**，它带 `"debugger": true` 元数据才支持调试。

**已装依赖**：`ms-python.python`、`ms-toolsai.jupyter`、`ms-python.debugpy 2026.6.0`、
venv 内 `debugpy 1.8.22` + `ipykernel 7.4.0` —— 调试所需的链路是完整的。

### 模型后端

书里所有示例默认 OpenAI / Gemini / Vertex。本学习环境改为 **DeepSeek 官方 API（OpenAI 兼容）**，书里的代码**无需改动**，只是 `base_url` / `model` 不同：

- `BOOK_BASE_URL` = `https://api.deepseek.com/v1`
- `BOOK_MODEL` = `deepseek-flash`（1M 上下文）
- `BOOK_MODEL_STRONG` = `deepseek-v4-pro`（Ch17 推理章用）

密钥存放于仓库根目录 `.env`（权限 600，`.gitignore` 里已改成 `.env*`，
连备份 `.env.bak-*` 一起排除，不会进 git）。

**`.env` 里有两处 key，指向同一个密钥**，两个都要改，别只改一处：

| 变量 | 谁在读 | 用途 |
|---|---|---|
| `BOOK_API_KEY` | `common/llm.py: make_llm()` | 本仓库脚本与 lab notebook |
| `OPENAI_API_KEY` | langchain-openai 自动读 | 书里**裸 `ChatOpenAI(...)`** 的原始代码 |

> 2026-11 起换成**本书学习专用的 DeepSeek key**（便于在平台上单独看用量，
> 不和工作其他调用混在一起）。改完实测三条路径都通：
> `make_llm()`、`make_llm(strong=True)`（`deepseek-v4-pro`）、裸 `ChatOpenAI()`。
> 想核对用量就去 DeepSeek 平台的用量页，这个 key 名下的调用全部来自本书学习。

**Gemini 侧同样有两套后端**（ADK 章节用），切换只改 `.env` 一行，代码不动
（`common/adk.py: adk_model()`）：

| `ADK_BACKEND` | 后端 | 限制 |
|---|---|---|
| `official` | 官方 AI Studio | **每模型 20 次/天**（免费层实测值，不是"每分钟"） |
| `relay`（当前） | xiaojingai 中转站（New API 网关） | 付费额度，无每日限制 |

中转站实测能力（2026-11）：`/v1beta` Gemini 原生格式 ✅、function calling ✅、
`google_search` ✅、ADK `LoopAgent` / `exit_loop` ✅。
实现上有两个坑：

1. `genai` SDK **不读** base_url 环境变量，必须显式构造
   `Gemini(model=..., base_url=..., client_kwargs={"api_key": ...})`。
2. **中转站必须绕开代理**。本机 `.bashrc` 有 `http_proxy=127.0.0.1:7897`，
   而中转站是国内服务，被绕去境外节点会出现
   `SSLError: DECRYPTION_FAILED_OR_BAD_RECORD_MAC`（随机失败、有时 4s 有时 40s）。
   已在 `../../common/adk.py` 里对中转站客户端强制 `trust_env=False` 并把 host 加进 `NO_PROXY`。
   验证方式：把 `https_proxy` 指到死端口再跑 notebook，三份都能跑完。

### 已验证跑通

书里 §1.3 的原版代码，**一字未改**，仅替换 LLM 初始化，脚本和 notebook 两条路径都跑通：

```bash
~/agentic_dev_venv/bin/python ch01_prompt_chaining/book_example.py
```

输出：
```json
{ "cpu": "3.5 GHz octa-core", "memory": "16GB", "storage": "1TB NVMe SSD" }
```

### 尚未安装（按需再装）

| 包 | 影响章节 | 为什么暂缓 |
|---|---|---|
| `google-adk` | 2,3,4,7,8,12,13,18 | 需 Google 凭证 / Vertex，本机跑不通，装了也只能读代码 |
| `crewai` | 5,6,7 | 依赖链很重（litellm 等），易与现有版本冲突 |
| `a2a-sdk` | 15 | 章节靠后，届时单独建环境更稳 |

### 目录约定

```
agentic_design_patterns/
├── .env                      # 后端配置（600，gitignore）
├── requirements.txt          # 人工维护的依赖清单
├── requirements.lock.txt     # pip freeze 全量快照
├── common/llm.py             # make_llm() 统一入口，全书练习都走这里
├── ../docs/google_credentials.md # §2.4 所需 Google 凭证的获取与配置
├── ch01_prompt_chaining/     # 第 1 章实践代码
└── Agentic-Design-Patterns/  # 克隆的书本仓库（PDF + notebooks + text/）
```

---
*本文件随学习进度更新。*

---

## 故障排查（实战踩坑记录）

### ① `ValidationError: Unknown scheme for proxy URL URL('socks://127.0.0.1:7897/')`

**现象**：裸 `ChatOpenAI(temperature=0)` 在**构造时**就报 pydantic ValidationError，
栈里全是 `langchain_core/load/serializable.py` → `pydantic/main.py`，看不出跟网络有关。

**根因链**（本机实测确认）：

```
GUI 会话导出 all_proxy=socks://127.0.0.1:7897/     ← mihomo/clash 写的
        ↓  （同时还有合法的 http_proxy=http://...，所以部分工具不受影响）
socks 不是标准 scheme，httpx 只认 socks5
        ↓
openai SDK 构造 httpx.Client 时【立即】校验所有代理 mount（不是发请求才校验）
        ↓
任何 ChatOpenAI(...) 实例化当场爆炸
        ↓
缺 socksio 时，即使写 socks5:// 也照样失败
```

**关键教训**：只改 `../../common/llm.py` **不够**。书里的原始 notebook 写的是裸
`ChatOpenAI(temperature=0)`，根本不经过我们的函数。必须**在进程启动阶段**解决。

**修复**：`../../tools/_book_bootstrap.py` + `../../tools/install_book_bootstrap.sh`，
安装成 site-packages 下的 `.pth`，该 venv 内**所有** Python 进程启动时自动：

1. `socks://` → `socks5://`，无法识别的 scheme 直接剔除
2. 加载学习用 `.env`（用 setdefault，不覆盖已有变量）

**踩过的弯路 —— 为什么不用 `sitecustomize.py`**：
Debian 自带 `/usr/lib/python3.12/sitecustomize.py`，而 `/usr/lib/python3.12`
在 `sys.path` 上**排在 site-packages 之前**，会把我们放的那份**静默遮蔽**掉，
放进去完全不生效（`import sitecustomize` 会指向 `/usr/lib/python3.12/...`）。
改用 `.pth`（`site.py` 会执行 `.pth` 里以 `import ` 开头的行），不受顺序影响。

已安装 `httpx[socks]`（`socksio 1.0.0`）提供 SOCKS 能力。端口 7897 是 mihomo 的
**mixed port**，HTTP 和 SOCKS5 都能通。

验证（clean env，只给 `ALL_PROXY=socks://`）：

| 场景 | 结果 |
|---|---|
| 裸 `ChatOpenAI(temperature=0)` 构造 | ✅ 代理报错消失 |
| 书里原版 notebook（仅补 model） | ✅ 0 错误 |
| `01_3_debug_lab.ipynb` | ✅ 0 错误 |
| `pip` | ✅ 正常 |
| 启动开销量级 | ✅ 0.00 s |

### ①-b 代理重新配置后的复核（2026-10-29）

用户重新配置了代理。实测复核：

| 检查点 | 结果 |
|---|---|
| GNOME 系统代理 | `manual`：http `127.0.0.1:7897`，socks `127.0.0.1:7897` |
| 当前运行中的 kernel | ✅ `ALL_PROXY=socks5://...`（已不是 `socks://`） |
| VS Code 主进程 / extension host | ⚠️ 仍带旧的 `ALL_PROXY=socks://...`（启动早于改配置） |
| `_book_bootstrap.py` 归一化 | ✅ 两个 venv 对 `socks://` 和 `socks5://` 都正确 |
| DeepSeek / Google 连通 | ✅ 直连、http 代理、socks5 三条路都通 |

**结论**：系统级已修好；仅残留老进程的旧环境变量。因为启动引导会
在 Python 启动时把 `socks://` 纠正为 `socks5://`，所以**不影响使用**。

> 想要彻底清爽：`Ctrl+Shift+P` → `Developer: Reload Window`，
> 让 VS Code 重新继承新环境。不做也可以。

**连通性实测值**（返回 401/403 = 已连到服务器，只是缺凭证）：

| 目标 | 直连 | http 代理 | socks5 |
|---|---|---|---|
| `api.deepseek.com` | 401 ✅ | 401 ✅ | 401 ✅ |
| `generativelanguage.googleapis.com` | 403 ✅ | 403 ✅ | 403 ✅ |

### ② 修完代理后的下一个坑：`gpt-3.5-turbo`

代理修好后，书里的裸 `ChatOpenAI(temperature=0)` 会换一种方式失败：

```
The supported API model names are deepseek-flash, deepseek-v4-pro,
but you passed gpt-3.5-turbo.
```

原因：`ChatOpenAI` 的 `model_name` 字段默认值**硬编码**为 `"gpt-3.5-turbo"`，
**没有对应的环境变量可以覆盖**（`base_url` 和 `api_key` 有，`model` 没有）。

**所以书里的代码不可能 100% 零改动跑在 DeepSeek 上。** 最小改动是补一个 `model`：

```python
# 书里原版
llm = ChatOpenAI(temperature=0)
# 改成一个词
llm = ChatOpenAI(temperature=0, model="deepseek-flash")
```

`OPENAI_API_KEY` / `OPENAI_API_BASE` 已写进 `.env`，所以不用再传 key 和 base_url。
或者直接用 `make_llm()`，连 model 都不用写：

```python
from common import make_llm
llm = make_llm(temperature=0)
```

### ③ 找不到 `agentic-book` kernel

**不影响使用。** `agentic_dev_venv` 和 `agentic-book` 指向同一个解释器
（`/home/stvli/agentic_dev_venv/bin/python`），调试能力完全一样。
上述修复是**环境级**的，跟选哪个 kernel 无关。

若想用 `agentic-book`：kernel 选择器 → **Select Another Kernel…** →
**Jupyter Kernel…** → `Python (agentic-book)`。

### 卸载本环境的启动引导

```bash
SITE=$(~/agentic_dev_venv/bin/python -c 'import site;print(site.getsitepackages()[0])')
rm -f "$SITE/_book_bootstrap.py" "$SITE/zz_book_bootstrap.pth"
```

---

## notebook 可移植性约定

所有练习 notebook 的 `common` 导入都写成**向上搜索**形式，不假设 kernel 的
工作目录：

```python
_root = Path.cwd().resolve()
for _d in (_root, *_root.parents):
    if (_d / "common").is_dir():
        sys.path.insert(0, str(_d))
        break
```

原因：VS Code 的 `jupyter.notebookFileRoot` 若被改动、或从命令行用
`jupyter lab` / `nbclient` 跑，cwd 可能不是仓库根，写死 `Path.cwd()` 会
`ModuleNotFoundError: No module named 'common'`。

已在**从 `chapter_notebooks/` 目录执行**这种最坏情况下验证通过：

| notebook | errors |
|---|---|
| `Chapter_01_Prompt_Chaining_(Code_Example).ipynb` | 0 |
| `Chapter_02_Routing_(LangGraph).ipynb` | 0 |
| `../../ch01_prompt_chaining/01_3_debug_lab.ipynb` | 0 |
| `../../ch02_routing/02_3_debug_lab.ipynb` | 0 |

### ③ 找不到 `agentic-adk` 内核（已定位）

**现象**：VS Code 的 kernel 选择器里看不到 `Python (agentic-adk)`；
即使 notebook 元数据写的是 `agentic-adk`，实际启动的却是 `agentic_dev_venv`。

**日志证据**（`~/.config/Code/logs/<ts>/window1/exthost/.../4-Jupyter.log`）：

```
Starting Kernel (Python Path: ~/agentic_dev_venv/bin/python, Venv, 3.12.3)
  for '.../Chapter_02_Routing_(Google_ADK).ipynb'
[warn] No interpreter with path ~/agentic_adk_venv/bin/python found in Python API,
       will convert Uri path to string as Id
```

**根因**：Python 扩展**不会扫描 $HOME 下的裸 venv**。`agentic_adk_venv` 不在
工作区内，也没被 `python.venvPath` 覆盖，于是 VS Code 的 Python API 不认识这个
解释器 → Jupyter 扩展无法把 `agentic-adk` kernelspec 映射成可用环境 → **静默回退**
到 `agentic_dev_venv`（它认识那个，因为设置里写了 `defaultInterpreterPath`）。

后果：跑 §2.4 时 `ModuleNotFoundError: No module named 'google'`。

**修复（第一次尝试，无效）**：

`python.venvPath` / `python.venvFolders` 是**旧版** `ms-python.python` 的键。
本机实际生效的是**新版** `ms-python.vscode-python-envs`，它**不读这两个键**，
所以加了也没用 —— 日志证实它只扫到了 `/usr/bin/python3` 和 conda 环境。

**修复（第二次，正确）**：

```jsonc
// 新版环境管理器读这个
"python-envs.globalSearchPaths": ["${env:HOME}"],
// 旧版仍保留（影响 .py 调试的解释器选择）
"python.venvPath": "${env:HOME}",
"python.venvFolders": ["agentic_dev_venv", "agentic_adk_venv"],
```

改完**必须 Reload Window**（`Developer: Reload Window`）才会重新扫描。

**若仍不出现**，退路是把 `google-adk` 直接装进 `agentic_dev_venv`，
用单一内核 `agentic-book` 跑全部章节。依赖安全性已验证 —— 唯一实际变化是
`websockets 17.3 → 15.0.1`，而所有相关约束都兼容：

| 包 | 约束 | 15.0.1 是否满足 |
|---|---|---|
| `openai`（realtime extra） | `<16,>=13` | ✅ |
| `langsmith` | `>=15.0` | ✅ |
| `langgraph-sdk` | `>=14` | ✅ |
| `fastmcp-slim` | `>=15.0.1` | ✅ |
| `google-adk` | `<16,>=15.0.1` | ✅ |

---

## ⚠️ 环境纪律（重要）

**`~/agentic_dev_venv` 是主环境，未经明确许可不要改动它。**

曾在未获同意的情况下把 `google-adk` 装进主环境，触发了
`websockets 17.3 → 15.0.1` 的降级。已按 `requirements.lock.txt` 完整还原：

| 检查 | 结果 |
|---|---|
| 与快照的额外包 | 仅 `socksio`（代理功能必需，刻意保留）|
| 版本差异 | **无**（websockets 17.3 / tenacity 9.2.1 / otel-api 1.45.1 均已还原）|
| `pip check` | No broken requirements |
| DeepSeek 调用 | ✅ PONG |
| `google.adk` / `google.genai` | ✅ 已移除 |

**Google 相关依赖只装在 `~/agentic_adk_venv`**，两个环境互不污染。

还原方法（万一再次被改动）：

```bash
# 以 requirements.lock.txt 为准做差异比对，再卸载多出的、还原变动的版本
# 注意保留 socksio
```