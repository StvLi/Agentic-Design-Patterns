# 笔记：多 agent 的顺序、并发与 state 语义

第 7 章的技术核心不是"有几个 agent"，而是**它们之间怎么传数据**。这份笔记把机制和坑记下来。

## 一、顺序是"串行等待"实现的，不是调度

`SequentialAgent._run_async_impl` 的核心就是一个 `for` 循环：

```python
for i in range(start_index, len(self.sub_agents)):
    sub_agent = self.sub_agents[i]
    async with Aclosing(sub_agent.run_async(ctx)) as agen:
        async for event in agen:       # 把这个子 agent 的事件流消费完
            yield event
# 循环结束才轮到下一个
```

**没有任何并发原语**，就是一个接一个 await 完 → 顺序天然成立，不需要额外保证。

`ParallelAgent` 用 anyio 的 task group：

```python
tasks.append(tg.create_task(process_an_agent(...)))   # 每个子 agent 一个 task
```

→ 完成顺序由**实际耗时**决定，与声明顺序无关。

## 二、state 的语义：单值覆盖、无历史、无所有权

`session.state` 是整条流水线**共享的全局命名空间**：

| 特性 | 现状 |
|---|---|
| 作用域 | 全局，任何 agent 可读写任意 key |
| 所有权 | 无记录（不知道谁写的） |
| 类型 | 不强制（实际都是字符串） |
| 历史 | **没有**，只保留当前值 |
| 冲突语义 | **last-write-wins** |

## 三、实测：多个 agent 写同一个 key，谁赢？

三个 agent 都设 `output_key="data"`，分别输出"苹果/香蕉/橘子"：

| 容器 | 声明顺序 | 实测 `state["data"]` |
|---|---|---|
| **Sequential** | A(苹果) → B(香蕉) → C(橘子) | `'橘子'` —— 最后一个，**确定** |
| **Parallel**（跑 3 次） | 同上 | `['香蕉', '苹果', '苹果']` —— **每次不一样** |

两个结论：

1. **语义是 last-write-wins**，不合并、不留历史。
2. **Sequential 下确定；Parallel 下是竞态** —— 并行那次没有一次是"橘子"（声明里的最后一个），
   说明它完全按完成时序，不是按声明顺序。
3. **ADK 不会警告你**两个 agent 抢同一个 key —— 静默的坑。

## 四、三种通信方式（本章六份脚本其实只用了这三招）

| 方式 | 怎么写 | 出现在 | 数据形式 |
|---|---|---|---|
| **共享 state** | A 设 `output_key="x"`，B 用 `{x}` 读 | Sequential / Parallel / Loop | 单值，覆盖 |
| **转交（handoff）** | 父 agent 设 `sub_agents=[...]`，模型自己选 | Coordinator | 整个对话上下文 |
| **当成工具调** | `AgentTool(agent=子agent)` | AgentTool | 子 agent 的返回值 |

三者的共同点：**全靠字符串名字维系**（state 键名 / 子 agent 的 `name` / 工具的 description）。
写错不会编译报错——第 2、3 章反复出现的那个问题，在多 agent 里被放大。

## 五、工程守则

1. **一个 agent 一个唯一 key**：`step1_data`、`weather_data`，别都用 `data`。
2. **并行分支之间绝不共享 key** —— 这是唯一会真的踩雷的地方（随机 bug，难复现）。
3. **要合并就设一个专门的汇合 agent**：读多个 key，输出一个新 key；
   而不是让多个 agent 争写同一个。
4. **`{key}` 占位符只认花括号**：写 `state['data']` 不会替换，模型只能看到那句英文描述，
   然后凭自己的知识编——**输出看起来完全正常**（最隐蔽的失败）。
5. **LlmAgent 不能直接写 state**：只能通过 `output_key`（写自己的输出）或工具
   （`tool_context.state["x"] = ...`）。书里 LoopAgent 那份就栽在这里。

## 六、和 ROS 概念的对照

如果拿 ROS 类比：

| | ROS topic | ROS 参数服务器 | ADK `session.state` |
|---|---|---|---|
| 多发布者 | 合法，订阅者收到**所有消息** | 合法但**互相覆盖** | 合法但**互相覆盖** |
| 历史 | 有消息序列 | 只有当前值 | 只有当前值 |
| 语义 | 广播 / 流 | last-write-wins | last-write-wins |

所以 `output_key` 更接近**参数服务器**，不是 topic。
想要"流"的效果，得自己在事件层记录（遍历 `event` 时保存），`session.state` 拿不到过程。

## 七、一句话

> **顺序靠串行 await 保证；并发允许，但 state 是"最后写赢"，所以并行时撞 key 就是随机 bug。**
