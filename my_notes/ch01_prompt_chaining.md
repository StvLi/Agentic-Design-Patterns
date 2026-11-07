# 第 1 章 · Prompt Chaining

对应 `chapter_notebooks/Chapter_01_Prompt_Chaining_(Code_Example).ipynb`。
这份笔记承接 notebook 里原来那一大段说明——**notebook 只留主线，细读在这里**。

## 一句话概括

把「从一段自然语言里抽出规格」和「把规格转成 JSON」拆成**两次独立的 LLM 调用**，
中间用**一个 dict 的键名**把两段接起来。

它做的**不是**"让模型一次输出 JSON"，而是"让模型分两次、每次只干一件事"。
这就是 Prompt Chaining 的全部主张。

## 执行时序：到底发生了什么

| # | 环节 | 类型 | 花 token | 输入 → 输出 |
|---|---|---|---|---|
| 1 | `prompt_extract` 插值 | 本地 | ❌ | `{"text_input": "..."}` → `ChatPromptValue` |
| 2 | `llm` 第 1 次调用 | **网络** | ✅ | `ChatPromptValue` → `AIMessage` |
| 3 | `StrOutputParser` | 本地 | ❌ | `AIMessage` → `TextAccessor`（str 子类） |
| 4 | dict 打包 | 本地 | ❌ | 字符串 → `{"specifications": "..."}` |
| 5 | `prompt_transform` 插值 | 本地 | ❌ | 同上 → `ChatPromptValue` |
| 6 | `llm` 第 2 次调用 | **网络** | ✅ | `ChatPromptValue` → `AIMessage` |
| 7 | `StrOutputParser` | 本地 | ❌ | `AIMessage` → `str` |

两个反直觉的结论：

1. **7 个环节里只有 2 个花 token。** 其余 5 个都是本地字符串/类型搬运。
   LangChain 的价值不在"让模型更聪明"，而在**把这 5 个搬运步骤标准化、可组合**。
2. **延迟 ≈ 2 × 单次模型调用。** 链每加一环（且含 LLM），延迟和成本线性叠加。

## 六个关键点（书里没明说，但决定你怎么用它）

### ① 惰性求值：`|` 不执行任何东西

```python
chain = a | b | c          # 零次调用
result = chain.invoke(x)   # 这时才开始跑
```

把"组装"和"执行"分开是 LCEL 的核心设计。好处：链可以复用、可以只跑前半截、
可以用 `.stream()` / `.batch()` 换执行方式而不用改组装逻辑。

### ② dict 就是「接口」，而且是个极脆弱的接口

```python
{"specifications": extraction_chain}
```

这一个 dict 承担了三件事：绑定变量名、声明数据流、隐式构造并行执行器。
但它**没有类型标注、没有 schema、没有校验**。上游吐什么，下游就信什么。

### ③ 全书最容易误解的一点：`final_result` 不是 JSON

它是 `str`。`print` 出来像 JSON，纯粹是模型配合表演。如果模型返回：

```
Here are the specs: cpu is 3.5GHz...
```

这个脚本照样跑完、照样打印、照样"成功"。

### ④ 花括号是保留字符

想在一个 prompt 里写字面量 `{"cpu": "..."}`，必须写成 `{{"cpu": "..."}}`。
这就是为什么本书这个 prompt 用 "with 'cpu', 'memory', and 'storage' as keys"
这种绕弯说法 —— 避开转义麻烦。

### ⑤ `temperature=0` ≠ 确定性

`deepseek-flash` 是带推理的模型（`usage_metadata` 里有 `reasoning` token 计数）。
推理路径本身带随机性，`temperature=0` 只压采样温度、冻不住推理。
实测中同一个输入曾给出 `"16GB"` 和 `"16 GB"` 两种结果。

### ⑥ 错误传播是单向且静默的

设每一步成功率为 p，链长为 n，整体近似 pⁿ。p=0.95、n=2 → 0.90。
**更糟的是错误不会被发现**：第 2 步无从知道第 1 步漏了信息。

举例：如果第 1 步漏掉了 "1TB NVMe SSD"，第 2 步**照样能编一个** `"storage": "1TB"`
出来 —— 格式完美、语气可信、内容错误。

> 所以后面必须引入校验步：第 4 章 Reflection、第 12 章异常处理、第 18 章 Guardrails。
> **Prompt chaining 提升的是可控性，不是正确性。**

## 如果想把这段代码「生产化」，缺什么

| 缺什么 | 后果 | 正解 |
|---|---|---|
| 输出校验 | 模型返回散文也算"成功" | `json.loads` + pydantic schema（第 18 章） |
| 结构化输出 | 靠 prompt 祈祷格式正确 | `with_structured_output()` / JSON mode（第 5 章） |
| 失败重试 | 一次网络抖动全链作废 | 指数退避（第 12 章） |
| 可观测性 | 出错时不知道哪一步坏的 | tracing / LangSmith（第 19 章） |
| 中间态持久化 | 重跑要从头再来 | LangGraph checkpoint（第 8 章） |

## 本仓库里的文件

- `book_example.py` —— 与书一致的脚本
- `01_3_debug_lab.ipynb` —— 拆成 9 格的双后端调试实验
