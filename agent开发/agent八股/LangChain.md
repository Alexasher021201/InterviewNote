可以从 **“它为什么出现 → 它有什么 → 怎么运行 → 和 Agent 什么关系”** 来理解 LangChain。

### 1. LangChain 是什么

**LangChain 是一个用于开发 LLM 应用和 AI Agent 的开源框架。**

如果不用 LangChain，你自己开发 Agent，可能要手动处理：

```text
用户问题
   ↓
拼 Prompt
   ↓
调用 LLM API
   ↓
解析模型输出
   ↓
判断是否调用 Tool
   ↓
执行 Tool
   ↓
把 Tool 结果重新交给 LLM
   ↓
生成最终答案
```

LangChain 把这些常见能力进行了**封装和标准化**，开发者可以更方便地把它们组合起来。

---

### 2. LangChain 的核心组件

你可以重点记下面几个：

| 组件 | 作用 | 举例 |
|---|---|---|
| **Model** | 调用大模型 | GPT、Claude、Gemini |
| **Prompt** | 管理输入给模型的提示词 | System Prompt、Prompt Template |
| **Tool** | 让模型使用外部能力 | 搜索、数据库、API |
| **Retriever** | 从知识库检索信息 | RAG |
| **Agent** | 让 LLM 决定下一步做什么 | 决定调用哪个 Tool |
| **Memory / State** | 保存任务过程中的上下文状态 | 对话历史、Agent 状态 |

所以可以简单记：

> **LangChain = Model + Prompt + Tool + RAG + Agent 等 AI 应用组件的开发框架。**

---

### 3. LangChain 最重要的能力：Tool Calling

例如用户问：

> “帮我查一下新加坡今天的天气，然后告诉我适不适合跑步。”

LLM 本身不知道实时天气。

于是我们可以给 Agent 注册一个：

```text
WeatherTool(city)
```

运行过程可能是：

```text
用户
 ↓
LangChain Agent
 ↓
LLM 分析问题
 ↓
决定调用 WeatherTool
 ↓
WeatherTool("Singapore")
 ↓
返回天气数据
 ↓
LLM 分析天气
 ↓
生成最终回答
```

这里非常重要的一点是：

**LangChain 本身不是大模型。**

它更像大模型外面的一层**应用开发框架 / 编排层**。

---

### 4. LangChain 和 RAG

LangChain 也经常用于构建 **RAG（Retrieval-Augmented Generation）**。

比如公司有 10,000 份内部文档：

```text
用户：
“公司的年假政策是什么？”

        ↓
Retriever 检索知识库
        ↓
找到相关公司文档
        ↓
把相关内容 + 用户问题
交给 LLM
        ↓
LLM 根据资料回答
```

LangChain 提供了 Retriever、Document Loader、Embedding、Vector Store 等相关抽象，因此很适合把整个 RAG Pipeline 串起来。

---

### 5. LangChain 和 Agent 的关系

这是 Agent 面试里比较值得理解的一点。

可以认为：

```text
LLM
= Agent 的“大脑”

Tool
= Agent 的“手”

Memory / State
= Agent 的“记忆”

LangChain / LangGraph
= 把这些东西组织起来的开发框架
```

例如一个数据库 Agent：

```text
用户
“帮我看看昨天订单为什么下降了。”

        ↓
      Agent
        ↓
   LLM 思考任务
        ↓
决定调用 SQL Tool
        ↓
查询数据库
        ↓
得到订单数据
        ↓
LLM 分析
        ↓
发现某地区订单下降
        ↓
可能继续调用 Tool
        ↓
最终给用户结论
```

所以你之前学习的 **Tool Use、MCP、Agent Harness** 和 LangChain 是可以联系起来的：

```text
                AI Agent
                   │
          ┌────────┴────────┐
          ↓                 ↓
         LLM              Tools
                            │
                         MCP Server
                            │
                  Database / API / Search

LangChain / LangGraph
负责把这些组件组织、编排起来
```

### 最后一句话记忆

> **LangChain 是一个 LLM 应用开发框架，它把 Model、Prompt、Tool、RAG、Agent 等常用能力标准化，让开发者更容易构建复杂的 AI 应用。**

如果你准备 **Agent 开发面试**，LangChain 不需要一开始死记大量 API，优先理解 **Agent、Tool Calling、RAG、State 以及 LangChain 与 LangGraph 的关系**，这几个概念更重要。