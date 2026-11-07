# 第 6 章 · Planning

书里两个脚本：一个 CrewAI 的"先计划再写"，一个 OpenAI Deep Research API 的调用。
对应 `chapter_notebooks/Chapter_06_Planning_(Code_Example).ipynb` 和
`(Deep_Research_API).ipynb`。

## 一句话

> **Planning 就是：把"做什么"和"怎么做"分开。** 你给目标，agent 自己发现步骤序列。

书里给的那个判断标准值得记住：

> 用不用 planning agent，只取决于一个问题：**"how" 是需要被发现的，还是已经知道的？**

- how 已知且可重复 → 用**固定工作流**，限制自治，换取可预测
- how 需要现场发现 → 才用 planning

这条判断比代码重要得多。因为"让 LLM 自己规划"的代价是**不可复现**——同一目标两次跑可能给出不同步骤序列。

## 两个脚本在演示什么

| notebook | 形态 | 能不能跑 |
|---|---|---|
| `(Code_Example)` | CrewAI：一个 agent，task 里写"① 列计划 ② 按计划写" | ✅ 已适配跑通 |
| `(Deep_Research_API)` | OpenAI Deep Research：托管的长流程研究 agent | ⛔ 需 OpenAI Deep Research 权限 |

### 必须先点破的一件事：书里这个例子是**弱规划**

```python
process=Process.sequential      # ← 顺序执行一个任务
```

`Process.sequential` 的意思是"任务按顺序跑"，**它本身不做任何规划**。真正的"规划"藏在哪？
藏在 `Task.description` 那两句话里：

```
1. Create a bullet-point plan for a summary on the topic: '...'
2. Write the summary based on your plan, keeping it around 200 words.
```

也就是说：**计划是模型在同一个回合里、在回答过程中顺手列出来的**。它没有落盘、没有经过审阅、
没有跨回合复用，也没法中途改。

这是"看起来像 planning，其实是 prompt 里加了句'先列提纲'"。

**真正的 planning 有三种更硬的形态**，书里在后面都提到了，只是没写全：

1. **拆成两个 task**：第一个 task 产出 plan（结构化、可检查），第二个 task 消费它 →
   这时计划才是一个**可见的中间产物**
2. **`Process.hierarchical`**：多一个 manager agent 负责分解与分配（CrewAI 原生支持）
3. **计划 → 人审 → 迭代执行**：Google DeepResearch 就是这么做的——它先把研究计划
   呈现给用户确认和修改，再进入"搜索 → 评估 → 补搜"的循环

> 所以这一节真正值得读的是**后半段**：Deep Research API 把 `reasoning` /
> `web_search_call` / `code_interpreter_call` 作为**结构化的一步一步**返回出来。
> 那才是"计划的可见形态"。前端那个 CrewAI 例子只教了"如何把计划塞进 prompt"。

## 一个小实验：persona 到底有没有用？

notebook 第二格做了对照：**任务描述和期望输出完全不变**，只把 agent 换成"几乎空白"
（`role='Writer'`，`goal` 和 `backstory` 都清空），各跑一次。

| | 完整人设 | 几乎无人设 |
|---|---|---|
| 计划条数 | 7 | 7 |
| 正文词数 | 165 | 122 |

**结论**：

1. **"要不要先列计划"完全不受 persona 影响** —— 两个 agent 都老老实实列了 7 条。
   真正起作用的是 `Task.description` 里那句 "1. Create a bullet-point plan..."。
2. 正文长度差了 43 词，但**单次运行无法归因**——也可能只是采样波动。
   要确认得跑 5 次取平均。

> 这条对写 agent 挺实用：**persona 不是控制手段，task 指令才是。**
> 你把人设写得多漂亮，都不如把"做什么"写清楚。

## 与前面章节的关系

| 章 | 提供的能力 | 在 planning 里的角色 |
|---|---|---|
| Ch1 Chaining | 固定顺序 | **how 已知**时，链就够了，不需要 planning |
| Ch2 Routing | 按内容选一条路 | 规划中的每一步都可能是一次路由 |
| Ch3 Parallelization | 同时走多条 | 计划里"这几步互不依赖"时才能并行 |
| Ch4 Reflection | 批评 → 改进 | 计划也需要被审（DeepResearch 让人来审） |
| Ch5 Tool Use | 能动手 | 计划的每一步最终都落成一次工具调用 |
| **Ch6 Planning** | **决定"按什么顺序做"** | 前面五章的编排者 |

一句话：**前五章是"怎么做好一步"，这一章是"怎么把多步排出来"。**

## 这一章的三个陷阱

1. **把"顺序执行"当成"规划"** —— `Process.sequential` 只是不并行，不是会规划。
2. **计划不可见** —— 藏在 prompt 里的计划无法检查、无法复用、无法中途修正。
   要让计划有用，它必须是**数据**（一个 list / 一个结构化对象），而不是一段 prose。
3. **过度规划** —— how 已知时还让模型自由发挥，等于用可复现性换灵活性。
   书里这句话说得很准：*"does the 'how' need to be discovered, or is it already known?"*

## 本机环境

| notebook | 该用哪个内核 | 说明 |
|---|---|---|
| `(Code_Example)` | `Python (agentic-crewai)` | 需要 crewai |
| `(Deep_Research_API)` | `Python (agentic-book)` | 只做结构演示，缺权限时不发请求 |

`(Code_Example)` 有两处【适配】：

1. **`llm` 的传法变了**，这一步踩了两个坑，都值得记：
   - 书里写 `ChatOpenAI(model="gpt-4-turbo")` 赋给 `Agent(llm=...)`，但 CrewAI 1.15 的
     `Agent.llm` 注解是 `str | BaseLLM | None`，**不再接受 LangChain 模型对象**。
   - 换成 `LLM(model="openai/deepseek-flash")` 也不行：CrewAI 1.15 改成**原生 provider**，
     `openai/` 前缀需要 litellm 回落，而 litellm 没随默认安装，于是报
     `ImportError: ... the LiteLLM fallback package is not installed`。
   - 正确写法是用内置的 `deepseek` provider（有独立前缀和默认 base_url）：

   ```python
   llm = LLM(model="deepseek/deepseek-flash",
             api_key=os.environ["OPENAI_API_KEY"],      # .env 里已有
             base_url="https://api.deepseek.com/v1")
   ```

   > 注意 `.env` 里的变量名叫 `OPENAI_API_KEY`，而原生 provider 默认找 `DEEPSEEK_API_KEY`，
   > 所以这里要显式 `api_key=` 传进去，免得为了一行代码再改一份环境。
2. **`kickoff()` 要放进独立线程**：VS Code 的 kernel 里有运行中的事件循环，
   CrewAI 的同步接口会直接抛 `RuntimeError`（和第 5 章 CrewAI 同一个坑）。
