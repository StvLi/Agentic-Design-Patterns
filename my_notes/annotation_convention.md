# 注释规范（增量式）

> 目的：让每一章的注释只解释**这一章新出现的东西**，读者可以按章累积地建立心智模型，
> 而不是每章都被重复解释一遍同样的组件。

---

## 一、总原则

**只注释「本章首次出现的模式相关组件」。**

- 之前章节已经讲过的组件 → **一行交叉引用**，不展开
- 本章新出现的组件/机制 → **展开讲透**（为什么这样设计、坑在哪）
- 纯语法噪音（`import`、`for` 循环、辅助函数）→ 不注释

判断标准是一句话：**读者读到这一章时，这一行代码是不是他第一次需要理解的东西？**

---

## 二、三级标注

| 级别 | 标记 | 用法 |
|---|---|---|
| **新** | `# ★ 本章新增：...` | 展开讲，可以多段 |
| **复用** | `# （Ch<N> 讲过）...` | 一行，指出在哪章出现过，不重复解释 |
| **无需** | 无注释 | 语言语法、显而易见的调用 |

外加一个**只用于环境差异**的标记：

| 标记 | 用于 | 例 |
|---|---|---|
| `【适配】` | 为了在本机跑起来而**偏离原书**的地方 | 模型名、`langchain_classic` 导入、`create_session_sync`、根目录 `sys.path` 搜索 |

`【适配】` 不承担解释职责，只做**可 diff 的标记**——读者一眼能看出"哪几行不是书里的"。
它不写长理由；理由（如果有必要）落到该章 `README.md`。

---

## 二·五、三种载体各放什么（重要）

早期版本把讲解堆在 notebook 的 markdown 格和代码横幅注释里，结果是
**「看原版很快懂用意，看改版经常 get lost」**。现在按载体分工：

| 载体 | 放什么 | 体量上限 |
|---|---|---|
| notebook 的 markdown 格 | 这个程序在干什么、kernel 选哪个、【适配】了哪些、细读去哪看 | **1 格，3–5 行** |
| notebook 的代码格 | 书里代码骨架原样保留 + `★` / `（ChN）` / `【适配】` 三种短注 | 注释**贴着那一行**，不写横幅段落 |
| `chN_*/README.md` | 执行时序表、关键点、实测记录、坑清单、生产化缺口 | 不限 |

判断标准：**读 notebook 时不该需要滚动。** 需要停下来细想的，去 README。

---

## 三、已讲组件台账（截至第 5 章）

后续章节遇到这些组件时，**只写交叉引用**。

| 组件 / 概念 | 首次出现 | 后续章节怎么写 |
|---|---|---|
| `ChatPromptTemplate.from_template` / `from_messages` | Ch1 | `# （Ch1）` |
| LCEL 的 `\|` 与惰性求值 | Ch1 | `# （Ch1：`\|` 只是组装，不执行）` |
| `StrOutputParser` | Ch1 | `# （Ch1）` |
| LCEL 里用 dict 做"交接边界" | Ch1 | `# （Ch1：dict 绑定变量名）` |
| `RunnableBranch` + 谓词 | Ch2 | `# （Ch2：显式路由）` |
| `RunnablePassthrough`（透传原始输入） | Ch2 | `# （Ch2）` |
| `RunnablePassthrough.assign`（追加键） | Ch2 | `# （Ch2：在输入上追加一个键）` |
| LLM-as-router / 标签分类 | Ch2 | `# （Ch2）` |
| ADK `output_key` → session state | Ch2 §2.4 | `# （Ch2 §2.4）` |
| ADK `sub_agents` / Auto-Flow 委派 | Ch2 §2.4 | `# （Ch2 §2.4）` |
| `RunnableParallel` + 扇出/扇合 | Ch3 | `# （Ch3）` |
| `asyncio` 并发 ≠ 并行（GIL） | Ch3 | `# （Ch3）` |
| ADK `ParallelAgent` / `SequentialAgent` | Ch3 | `# （Ch3）` |
| `@tool` / docstring 即工具描述 | Ch5 | `# （Ch5）` |
| `bind_tools` + `tool_calls` + `ToolMessage` | Ch5 | `# （Ch5）` |
| `args_schema` 参数约束 | Ch5 | `# （Ch5）` |
| notebook 里 `asyncio.run` 要用线程包 | Ch3 | `# （Ch3：线程内跑独立事件循环）` |

---

## 四、每章的"新增清单"（写注释前先列）

写的时候先问：这一章新出现的模式组件是什么？只有这些才展开。

### Ch1 Prompt Chaining
- LCEL 组装（`|`）
- 惰性求值：组装 ≠ 执行
- dict 作为两段之间的交接边界

### Ch2 Routing
- LLM-as-router：输出空间必须有限、互斥、单 token
- `RunnableBranch` 的谓词结构
- ADK Auto-Flow：`sub_agents` + `description` 驱动委派
- `output_key` → session state
- 路由错误的静默性

### Ch3 Parallelization
- `RunnableParallel`：扇出（submit 循环）+ 扇合（`future.result()` 字典）
- 并发 ≠ 并行（I/O vs CPU）
- all-or-nothing 失败语义
- ADK `ParallelAgent` / `SequentialAgent` 的声明式拓扑

### Ch4 Reflection ← **本章新增**
- **反思的三段结构**：生成 → 批判 → 精修
- **Producer / Critic 分离**（两个 prompt 或两个 agent）
- **循环 + 停止条件**：`CODE_IS_PERFECT` 协议，以及"精确匹配 vs 子串"的坑
- **对话历史即记忆**：`message_history` 让模型看到自己上一版
- **批判者用独立 messages**：避免锚定
- **ADK `inject_session_state` 只认 `{变量}`**：文字描述不会注入 state（原书 bug）
- **`is_final_response()` 在 `SequentialAgent` 下每个子 agent 都为真**

### Ch5 Tool Use
- `@tool` 装饰器：name/description/args_schema 的来源
- 工具调用循环：模型只"请求"，代码负责执行
- `tool_call_id` 配对
- 参数约束（`Field(ge=, le=)`）+ 模型静默钳制

---

## 五、反例（不要这样写）

```python
# ❌ 过度：每章都重讲一遍
# 这里的 | 是 LCEL 的管道运算符，它把前一个组件的输出传给后一个……
chain = prompt | llm | parser

# ✅ 增量：只提一次，之后交叉引用
# （Ch1：`|` 只是组装，不执行）
chain = prompt | llm | parser
```

```python
# ❌ 复述代码
# 调用 llm 的 invoke 方法，传入 messages
response = llm.invoke(messages)

# ✅ 讲"为什么"
# ★ 本章新增：模型回答也要 append 进历史，否则下一轮它看不到自己写过什么
message_history.append(response)
```

---

## 六、检查清单

写完一章的注释，自查四项：

1. 本章新增的组件，是否都标了 `★ 本章新增`？
2. 复用的组件，是否都只留了一行 `（Ch<N>）` 而没有重复解释？
3. 有没有"复述代码"式的注释（说 what 而不是 why）？
4. 坑是否写清了**现象**（不只是"要注意"）？
