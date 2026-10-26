# 第5章：把工作交给子 Agent

> **一句话**：主 Agent 不该什么活都自己干。派出去的子 Agent 在自己的上下文里干完，只把结论递回来——过程不进主 Agent 的消息，文件却是共用的。
>
> **本篇环境**：同前几篇。`deepagents==0.7.8`，`FilesystemBackend` 指向 `workspace/`。

---

## 一、本章任务

Task4：把一项工作交给子 Agent。学第 5 章和配套 Notebook，修改子 Agent 的职责、提示词或工具映射。

我先做了三件事：读章节、读配套 Notebook、把项目里现在的子 Agent 配置打出来看清楚。

---

## 二、第 5 章讲的是什么

整章就围绕一个词：**Context Quarantine（上下文隔离）**。

流程很简单：

- 主 Agent 通过 `task` 工具派活
- 子 Agent 在独立上下文中执行
- 执行完只把最终结果交回来
- 中间的搜索、读文件、写文件，一条都不进主 Agent 的消息

为什么要这样：一个子任务可能包含十几次工具调用，全堆在主 Agent 的上下文里，还没写到结论就满了。第 3 章的自动卸载能缓解，但治不了根——**主 Agent 本来就不需要知道这些细节**。

打个比方，主 Agent 是项目经理，子 Agent 是专项负责人。项目经理不参加每场技术讨论，只看负责人交上来的总结。

### 定义方式

最常见的是字典，三个必填字段：

| 字段 | 作用 |
|---|---|
| `name` | 主 Agent 用它指定派给谁 |
| `description` | 主 Agent 靠它决定**什么时候**派给谁 |
| `system_prompt` | 指导子 Agent 自己怎么干 |

可选的有 `tools`、`model`、`middleware`、`response_format`、`skills` 等。

### 两条容易记错的继承规则

- **`tools` 默认继承主 Agent，一旦显式写就完全替换，不合并。**
- **`system_prompt`、`middleware`、`skills` 都不继承。**

第二条我第 4 章已经碰过一次：子 Agent 不会自动带上 TodoListMiddleware。


---

## 三、我项目现在的样子

跑 `scripts/check_subagents.py` 打出来的（这个脚本只读图，不调模型）：

```text
main agent tools (14):
  ['delete', 'edit_file', 'execute', 'get_current_time', 'get_weather', 'glob',
   'grep', 'ls', 'read_file', 'task', 'tavily_search', 'think_tool',
   'write_file', 'write_todos']
  tavily_search on main agent: True

sub-agents advertised by the task tool:
  - general-purpose: … This agent has access to all tools as the main agent.
  - research-agent: Delegate research to the sub-agent researcher.
                    Only give this researcher one topic at a time.
```

看出来的两个问题：

**一、主 Agent 自己就有 `tavily_search`。** `prompts.py` 里白纸黑字写着 "ALWAYS use sub-agents for research, never conduct research yourself"，但工具在它手上。Task3 那次就是证据：它没委派，自己把三份资料读完、自己写了报告。

**二、`research-agent` 的描述几乎不起作用。** 那句 "Only give this researcher one topic at a time" 是在约束主 Agent **怎么用**，不是在说明它**擅长什么**。而 `general-purpose` 排在前面、能力更全，路由自然往那边偏。

---

## 四、配套 Notebook 里最值得看的一段

Notebook 是 offline 模式，用脚本模型跑，不需要 API key。它的实验只回答两个问题。

**问题一：主 Agent 看得到子 Agent 的全过程吗？**

主 Agent 的消息只有 4 条：

```text
human  请把 cache-plan 交给 researcher…
ai     调用 task(subagent_type="researcher")
tool   「方案 B 满足约束，报告位于 /research/cache-choice.md。」
ai     转述给用户
```

而 researcher 内部，其实调用了 `lookup_case` 读资料、`write_file` 写报告。Notebook 里直接断言了：主 Agent 收到的工具结果名是 `['task']`，**没有** `lookup_case`，也**没有** `write_file`。

**问题二：子 Agent 写的文件，主 Agent 拿得到吗？**

拿得到。报告躺在共享状态的 `files` 里，Python 能读、主 Agent 也能读——**但要它自己 `read_file` 才会进消息**。

所以那句话要记准：

> 消息隔离 ≠ 文件隔离。

---

## 五、我打算怎么改

三处，正好对应作业里说的「职责、提示词、工具映射」。

### 1. 职责：加一个专门看资料的

```python
materials_analyst = {
    "name": "materials-analyst",
    "description": (
        "分析 /materials/ 下的固定资料时使用。不需要联网，"
        "只读文件、提炼要点、把摘要写成文件。每次只给它一组指定资料。"
    ),
    "system_prompt": MATERIALS_ANALYST_INSTRUCTIONS,
    "tools": [],          # 显式空列表：不给自定义工具，只留框架自带的文件工具
}
```

同时把 `research-agent` 的 description 收窄成「需要联网检索最新信息时使用」，两个 Agent 的适用场景就分开了。

### 2. 工具映射：主 Agent 拿掉 `tavily_search`

```python
tools=[think_tool, get_current_time, get_weather]
```

这一步是关键。工具在，规则就只是建议；工具没了，委派才成为唯一的路。

### 3. 提示词：给返回结果上约束

两个子 Agent 都补上：

```text
输出格式：
1. 结论（不超过 100 字）
2. 证据标签（[1][2] 对应资料编号）
3. 产出文件路径

原始资料先用 write_file 落到文件，只把摘要返回。
不要复制原文，不要复述工具调用过程。
```

按第 5 章的说法：子 Agent 返回一大段原始数据，上下文隔离就白做了。

---

## 六、为什么这么改

对着章节里的五条最佳实践：

| 实践 | 我对应的改动 |
|---|---|
| 1. 描述要具体 | 两个子 Agent 的 description 写成「什么时候用它」，不是「我是什么」 |
| 2. 提示词要详细 | 补输出格式、字数上限、文件路径要求 |
| 3. 工具集要精简 | 主 Agent 去掉联网；资料分析员不给联网 |
| 4. 不同子 Agent 用不同模型 | 这次不做，一个模型够用 |
| 5. 返回结果要精简 | 结论 100 字 + 文件路径 |

`general-purpose` 我打算留着不动，用描述和主提示词把路由约束住。真要关它得走 `register_harness_profile`，多一层风险，暂时不碰。

---

## 七、和前面几章对得上的地方

- **第 3 章**：子 Agent 共用同一个 `FilesystemBackend`，所以它写的文件会落到 `workspace/`。这条正是「文件共享」那一半。
- **第 4 章**：子 Agent 不继承 TodoListMiddleware。它有自己的清单要单独开，主 Agent 的 `todos` 它也不碰。
- **第 6 章**：异步子 Agent 是另一套东西（要 Agent Protocol 服务承后台任务），这次不做。

---

## 八、本章完成结果

- 读完第 5 章，弄清 Context Quarantine：只回结论，不回过程
- 读了配套 Notebook 的 offline 实验，看清「消息隔离 + 文件共享」这两件事同时成立

---

## 参考

- 教程第 5 章：[子 Agent 与上下文隔离](https://datawhalechina.github.io/deepagents-in-action/chapters/ch05-subagents/)
- 配套实验：[第 5 章 Notebook：子 Agent 委派与上下文隔离](https://github.com/datawhalechina/deepagents-in-action/blob/main/notebooks/ch05/01-subagent-delegation.ipynb)
- 课程仓库：[datawhalechina/deepagents-in-action](https://github.com/datawhalechina/deepagents-in-action)
