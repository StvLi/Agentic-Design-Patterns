# 第 4 章 Reflection —— 章节导读

> 配套材料：`book_example.py`（可跑脚本）、`04_3_debug_lab.ipynb`（实验）、
> `local_llm.py`（本地小模型接入）
>
> 阅读顺序建议：**本文件 → book_example.py → 04_3_debug_lab.ipynb**

---

## 一、这一章解决什么问题

前三章的原子操作都是**一次性**的：

| 章 | 模式 | 控制流 | 输出之后 |
|---|---|---|---|
| 1 | Chaining | 直线 | 直接交付 |
| 2 | Routing | 分岔 | 直接交付 |
| 3 | Parallelization | 扇出 + 汇合 | 直接交付 |
| **4** | **Reflection** | **回环** | **先自审，再改进** |

前三章假设"生成出来的东西够好"。Reflection 承认一个朴素事实：

> **模型第一次给的东西经常不够好，而且它自己不知道。**

所以引入一个环：**生成 → 批判 → 精修 →（再看一遍）**。

这是本书第一次出现"回头"的控制流。也是为什么书里说"真正的迭代反思需要状态管理"
——环要记住上一轮做了什么（第 8 章会展开）。

---

## 二、核心结构：Producer / Critic

书里最重要的一句话是把两个角色**分开**：

```
        ┌────────────────────────────────────────┐
        │                                        │
  需求 ──┴─→ Producer ──→ 初稿 ──→ Critic ──→ 意见
   ↑                 ↑                     │
   │                 └───── Refine ←────────┘
   │                          │
   └────────── 循环 ───────────┘
```

- **Producer**：只管生成，不管对错
- **Critic**：只管挑毛病，给结构化反馈
- **分离的理由**（书里原话）：避免"自己审自己"的认知偏差

关键实现细节：**两个角色可以是同一个模型**，靠**不同的 system prompt** 区分。
这就是书里说的 "two separate LLM calls with distinct system prompts"。

---

## 三、本章材料怎么用

| 文件 | 内容 | 什么时候看 |
|---|---|---|
| `book_example.py` | 书里 §4.3 的单轮反思链（LCEL 实现） | 先跑一遍，看形状 |
| `04_3_debug_lab.ipynb` | **7 节实验**，含双后端对比 | 核心，逐节读 |
| `local_llm.py` | 本地 Qwen3.5-0.8B 接入 | 想复现实验时 |

### 为什么这个 lab 要接本地小模型

用 DeepSeek-V4.1-Flash 做反思实验有个尴尬：**基线一次就满分，反思没东西可改**。

实测（同一任务，10 条规格 14 项检查）：

| 后端 | 基线 | 反思后（拿到强批判） | 失败特征 |
|---|---|---|---|
| DeepSeek-V4.1-Flash | **14/14** | — | 无 |
| Qwen3.5-9B（Q8_0，本机 GPU） | **9/14** | **13/14** ✅ | 过度补零 + 漏中间零组件 |
| Qwen3.5-4B | **9/14** | 9/14 | 过度补零 + 漏中间零组件 |
| Qwen3.5-0.8B | **8/14** | 8/14 | **完全不取余**（`1h 60m`） |

**关键分水岭在 4B→9B**：9B 拿到强批判后真的把代码改对了（9→13），
4B 和 0.8B 都只改机械项、回避重构。

**两种后端下反思都没提升**——但原因完全不同，下面第四节解释。

切换后端：改 `04_3_debug_lab.ipynb` 第 0 格的 `BACKEND`。

---

## 四、本章最重要的一次测量：批判者的水平决定一切

lab 里做了一个受控实验：**人为往正确代码里注入已知缺陷**，看批判者能否发现。

结果（同一批代码、同一份规格，只换批判者）：

| 注入的缺陷 | 客观评分 | 弱批判者（0.8B） | 强批判者（DeepSeek） |
|---|---|---|---|
| 缺 `TypeError` 检查 | 8/14 | ❌ 说"完美" | ✅ 指出 |
| 缺 docstring 的 `Example` | 7/14 | ❌ 说"完美" | ✅ 指出 |
| 不补零（格式漏规格） | 10/14 | ❌ 说"完美" | ✅ 指出 |
| **命中率** | — | **0/3** | **3/3** |

**0.8B 当批判者，三个明显缺陷一个都没发现，还都回了 `CODE_IS_PERFECT`。**

> 后续补测（同一份 9/14 代码）：
> - **4B 当批判者：说 `CODE_IS_PERFECT`**
> - **9B 当批判者：能指出部分真实缺陷，但会误读规格、长篇自我纠结，有时仍以 PERFECT 收尾**
>
> 也就是说 0.8B → 4B 没跨过"能当批判者"的门槛；9B 只跨了一半；
> **只有 DeepSeek 是稳定可靠的批判者。**

于是整个循环在第一轮就停了：

```
轮次 0（生成）  8/14   调用 1
轮次 1: 批判者说 CODE_IS_PERFECT，提前停止
```

基线只有 8/14，反思却认为"已经完美"——**反思不但没帮忙，还给了错误的安全感。**

### 由此得出两个结论

**①-b 但能力是【不对称】的：9B 能"改"，不能"判"**

这是本次最重要的发现。同一台机器上：

| 角色 | 0.8B | 4B | 9B | DeepSeek |
|---|---|---|---|---|
| 当**生产者**（能按批判改代码） | ❌ | ❌ | **✅ 9→13** | ✅ |
| 当**批判者**（能发现缺陷） | ❌ | ❌ | ⚠️ 半可靠 | ✅ |

所以下面这个非对称组合是划算的：

```
9B 本地（免费、快、隐私）  →  生产 + 执行修改
DeepSeek（一次调用）      →  只做批判
```

实测：**9B 生产 + DeepSeek 批判 = 13/14**，而全用 DeepSeek 是 14/14。
**用 1/10 的成本拿到 93% 的效果**——这才是本地部署 + 云批判的真正意义。

**① 批判者比生产者更重要（反直觉）**

大家总在纠结"该用多强的模型来写"。实验说得很清楚：
**生产者弱一点没关系，批判者弱就全盘失效。**

一个实用的架构是**非对称**的：

```
弱模型（便宜 / 本地）  →  生产
强模型（贵 / 调用少）  →  批判      ← 每轮只调一次，成本可控
客观测试               →  兜底硬性正确性
```

批判者调用次数是生产者的 1/N（N=轮数），所以**用强模型做批判者其实很划算**。

**② 有客观评估器时，别指望 LLM 自评**

lab 的"客观评分"列能精确定位每个缺陷——因为它真的跑了代码。
更稳的循环是：

```
生成 → 跑测试 → 不达标 → 强模型批判（提示改哪里）→ 精修 → 再跑测试
```

**测试是裁判，LLM 批判只是辅助。**

---

## 五、三种停止条件

反思必须有退出条件，否则烧钱。

| 策略 | 写法 | 优点 | 风险 |
|---|---|---|---|
| 固定轮数 | `for i in range(N)` | 成本可预测 | 可能白跑 |
| 信号词 | 批判者输出 `CODE_IS_PERFECT` | 省调用 | **小模型会过早说"完美"**（实测） |
| 客观阈值 | 测试达到 N 分才停 | **最可靠** | 需要可执行评估器 |

实测结论：**信号词在小模型上不可靠**，第三种才是生产级方案。

---

## 六、成本：一圈 = 两次调用

书里提到反思更贵，但漏算了一点：

```
一轮反思 = 批判（1 次）+ 精修（1 次）= 2 次调用
总调用 = 1（生成）+ 2 × 轮数
```

而且每一轮 history 都在增长，后续调用的 **token 数更大**。
书里也提醒了："with each iteration, the conversational history expands"。

对策：只带**最近一版代码** + 累积的**已解决问题清单**，而不是全量历史。

---

## 七、三种失败模式

**① Critic 幻觉** —— 无中生有报 bug，或漏掉真 bug（上面 0/3 就是极端案例）

**② 越改越差** —— 反思**不保证单调改进**。生产上必须保留最好的一版：

```python
best_round = max(trace, key=lambda x: x[1])    # 取分数最高那轮，不是最后一轮
```

**③ 上下文膨胀** —— 轮次一多，成本上升且模型开始"忘记"最初需求。

---

## 八、生产清单

| # | 事项 | 说明 |
|---|---|---|
| 1 | **必须有客观评估器** | 光靠 LLM 自评不可靠 |
| 2 | **保留最好的一版** | 不保证单调改进 |
| 3 | **设硬上限** | 轮数 / token / 时间任一超限就停 |
| 4 | **记录每轮分数** | 出事时才知道哪轮坏的 |
| 5 | **控制上下文** | 只带最近版本 |
| 6 | **评估与生成分离** | 最好用不同模型 |
| 7 | **算清成本** | 一轮 = 2 次调用，token 更大 |
| 8 | **简单任务别用反思** | 实测强模型基线就满分，纯亏 |

---

## 九、一句话总结

> **Reflection 不是"让模型更聪明"，而是"把评审流程自动化"。**
>
> 它有效的前提是：**你有一个能判断好坏的评估器**。
> 没有评估器的反思，只是让模型多改几遍——可能变好，也可能变坏，
> 而且你可能根本不知道是哪一种。

---

## 十、本地小模型环境（可选）

实验用到的 Ollama 是**用户级安装**，不碰系统、不改主环境：

```bash
# 启动服务（后台常驻）
~/.local/ollama/bin/ollama serve &

# 确认模型在
~/.local/ollama/bin/ollama list
# → qwen3.5-0.8b:latest   532 MB

# 验证接入
~/agentic_dev_venv/bin/python ch04_reflection/local_llm.py
```

几个踩过的坑，记在这里省得重复：

| 坑 | 现象 | 解法 |
|---|---|---|
| Ollama 官方 `pull` 失败 | `redirect target not allowed ... resolves to non-public` | 本机透明代理把 CDN 解析成 198.19.x，Ollama 的 SSRF 防护拒绝跟随。改用 HF 下载 GGUF + `ollama create` |
| Qwen3.5-0.8B 返回空字符串 | `done_reason: length`，内容全在 `thinking` 字段 | 它是**思考型模型**，0.8B 会把 token 全花在思考上。必须 `think: false` |
| `/v1` 端点不认 `think:false` | 传了无效，仍返回空 | 只有原生 `/api/chat` 支持，所以 `local_llm.py` 直连它 |
| Modelfile 的 `TEMPLATE` 写 Jinja | `template error: function "content" not defined` | Modelfile 只支持 **Go 模板**；Jinja 只从 GGUF 元数据里读 |

---

## 十一、延伸：这一章在全书的位置

- **前置**：Ch1-3（链 / 分岔 / 扇出）——反思循环内部就是一条链
- **后续**：
  - Ch8 记忆管理 —— 反思需要记住上一轮，state 是基础
  - Ch12 异常处理 —— 反思是"软错误"的处理手段
  - **Ch18 Guardrails** —— 反思的反面：有些错误不该靠模型自审兜住
  - **Ch19 评测** —— 反思的地基，这章讲怎么造"评估器"

> 读到这里可以记住一个判断：
> **当你能写出评估器时，用测试；写不出评估器时，才用 LLM 当批判者——而且要用强的。**

---

## 十二、ADK 版：把批评接回生成端

对应的 notebook 是 `chapter_notebooks/Chapter_04_Reflection_(ADK).ipynb`。

书里这份是**顺序管道**：`SequentialAgent` 里 DraftWriter 写、FactChecker 查，跑完就结束。
批评只被打印出来，**从未回流到生成端**——那是「生成 + 检查」，不是 reflection。
本章讲的回路（批评 → 改写 → 再批评）在书里那版是缺席的。

### 补了什么

| # | 补的东西 | 为什么 |
|---|---|---|
| 1 | `LoopAgent(sub_agents=[writer, critic], max_iterations=3)` | 回路本身。state 是两端的唯一通道：`output_key` 写、`{占位符}` 读 |
| 2 | 生成端 instruction 加 `{draft_text}` 和 `{critique}` | 第 2 轮起是**改写**自己上一稿，而不是重新写一篇。**这两个占位符就是"回路的入口"** |
| 3 | 批评端加 `exit_loop` 工具 | 让它能**喊停**。没有这个工具，循环只能靠 `max_iterations` 硬停 |

还有一个不显眼但决定成败的点：**验收标准（spec）只放在批评端**。

原书两端都没有可检查的条目，批评端于是只能说"挺好"。实测把 spec 同时给两端时，
第一稿就 4 条全满足（连"莫泊桑"都写出来了）、直接 `APPROVED`，**回路空转**。
所以改成真实分工：生成端只拿 brief，批评端拿 spec —— 相当于**作者按 brief 写、
评审按测试套件查**。

### 实测轨迹：3 轮才收敛，中间出现一次回归

`gemini-3-flash-preview`，同一个主题，6 个 agent 回合：

| 轮 | 谁 | 发生了什么 |
|---|---|---|
| 1 | DraftWriter | 首稿：设计师 ✓、1889 ✓、330 米（**缺英尺**）、批评者写成"many"（**缺具体人名**） |
| 2 | FactChecker | 抓到 req 3（缺英尺）和 req 4（没写是谁批评的） |
| 3 | DraftWriter | **修好了这两条**，但同时把"设计师 Gustave Eiffel"改没了 |
| 4 | FactChecker | 抓到 **req 1 回归**：草稿里没有设计师的名字 |
| 5 | DraftWriter | 补回"Named after its designer, Gustave Eiffel"，保留前两轮的所有修正 |
| 6 | FactChecker | `APPROVED` + 调 `exit_loop` → 循环提前结束 |

**第 3→4 步才是这段最值得看的地方**：修一个坏一个。这是反思回路最典型的病理，
而且它说明批评端不能只检查"我说的问题改了没"，必须**每轮重新对照完整 spec** ——
否则回归会一路带着走到最后。

也顺便解释了为什么 `max_iterations=3` 是合理的：这个任务刚好要 3 轮。
调成 2 就会被硬停在一个仍有缺条的稿子上。

### 两个 ADK 特有的坑

1. **`exit_loop` 那一轮的事件文本是空的**
   `exit_loop` 会把 `skip_summarization` 置为 True，所以批准的那一轮
   `event.content` 的文本被抑制成空串。结果要从 `session.state['critique']` 读，
   不能指望事件流。notebook 里对这种情况打了显式提示。

2. **`state['critique']` 会停在上一轮**（这个坑更隐蔽）
   调 `exit_loop` 的那一轮 `skip_summarization=True` 抑制了文本输出，`output_key`
   于是可能没写进去，`session.state['critique']` 停在上一次的批评上。
   所以判断「是否批准」要看**回合数**（本次 4 < 2×3），不能看 state 内容。
   notebook 里对这种情况在标题上直接标了出来，避免误读。

3. **state 里缺 key 时占位符填不进去**
   第一轮跑时 `critique` / `draft_text` 还不存在，所以建 session 时要把三个键
   都先塞空串：`create_session_sync(..., state={"subject": ..., "draft_text": "", "critique": ""})`。
   这和 Ch3 讲的「契约靠字符串名字维系、没有任何静态校验」是同一件事。

### 顺带查清的配额真相

跑这一章时连续撞 429，查下去发现**不是每分钟限流**，而是：

```
quotaId: GenerateRequestsPerDayPerProjectPerModel-FreeTier
quotaValue: 20
retryDelay: 5661s
```

免费层是 **每个模型每天 20 次**，而且 **每个模型独立计数**（换模型 = 换桶）。
所以 notebook 里的错误处理分了两种：

- `PerMinute` / 503 / **TLS 抖动** → 瞬时，sleep 后重试
- `PerDay` → 当天用完了，**重试没意义**，直接报清楚并建议换桶

分类函数在 `common/adk.py`（`is_transient_error` / `is_daily_quota_exhausted` /
`retry_delay_seconds` / `short_error`），三份 ADK notebook 共用同一套，
免得各写一份慢慢走样。

实测当时还有额度的桶：`gemini-3-flash-preview`、`gemini-3.1-flash-lite`；
`gemini-3.5-flash` 和 `gemini-3.6-flash` 都已用完。


### 后端：官方 AI Studio ↔ 中转站

这一节跑一次要 4~6 次模型调用，而官方免费层是**每模型 20 次/天**，很容易见底。
`.env` 里留了一个开关（代码不用改，`common/adk.py` 负责）：

```
ADK_BACKEND=relay     # 中转站（New API 网关），付费额度、无每日限制
ADK_BACKEND=official  # 官方 AI Studio
```

关键实现细节：`genai` SDK **不读** base_url 环境变量（只认 GOOGLE_API_KEY /
GEMINI_API_KEY / GOOGLE_GENAI_USE_VERTEXAI 等），所以要指向中转站必须显式构造
`Gemini(model=..., base_url=..., client_kwargs={"api_key": ...})`。
用 `client_kwargs` 传 key 而不是覆盖 `GOOGLE_API_KEY`，两种后端才能在同一进程里共存。

### ⚠️ 代理必须绕开，否则 TLS 会随机炸

本机 `.bashrc` 里设了 `http_proxy` / `https_proxy = 127.0.0.1:7897`（Clash 之类）。
中转站 `open.xiaojingai.com` 是**国内服务**，被绕去境外节点后会出现：

```
SSLError: [SSL: DECRYPTION_FAILED_OR_BAD_RECORD_MAC] decryption failed or bad record mac
```

症状很迷惑：**不是每次都失败**，往往跑到一半（比如第 2 个 agent）才炸；
而且同样的调用有时 4 秒、有时 40 秒。原因是代理链路把 TLS 记录破坏了。

修法在 `common/adk.py`，两件事一起做：

```python
http_options = {
    "timeout": 60000,                                  # 毫秒
    "client_args":       {"trust_env": False},         # 别读 http_proxy / https_proxy
    "async_client_args": {"trust_env": False},
}
Gemini(model=..., base_url=..., client_kwargs={"api_key": ..., "http_options": http_options})
```

1. `trust_env=False`：中转站的客户端**强制不走代理**（官方 Google API 仍然可以走，互不影响）。
2. 把该 host 追加进 `NO_PROXY` 兜底（万一别处直接用 httpx）。

> 验证方式：把 `https_proxy` 指向一个**死端口**（如 `http://127.0.0.1:1`）再跑 notebook。
> 三份 ADK notebook 都能正常跑完，就说明确实绕开了代理。
>
> 另外，TLS 抖动这类错误已经纳入 notebook 的退避重试（`is_transient_error()` 里
> 包含 `SSL` / `DECRYPTION_FAILED` / `Timeout` / `ConnectError` 等），
> 所以即使偶发一次也会自动重试而不是整轮报废。
