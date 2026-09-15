可以。这一节比前面的「基础概念与架构模式」更偏**工程实践**。网页列了约 29 个问题，从 Function Calling 一直问到 MCP、Skills、大规模工具管理、权限、安全和异常处理；网页也明确把「Function Calling、MCP、Skills」列为 AI Agent / 应用开发岗的重点。

我还是按**面试时能直接说出口**的方式给你整理，并把容易连续追问的问题合并起来。

---

# 一、Function Calling 是什么？原理是什么？

对应网页第一题：

> Function Calling 的原理是什么？模型如何知道该调用哪个工具？如何理解 Tool Use？

### 标准回答

Function Calling 本质上是：

> **开发人员把可用工具的描述和参数 Schema 提供给大模型，大模型根据用户需求决定是否需要调用工具；如果需要，就生成结构化的工具名称和调用参数，最后由 Runtime / 应用程序真正执行工具。真正执行工具的是 Agent Runtime / 应用程序，而不是 LLM 自己。**

比如注册一个天气工具：

```json
{
  "name": "get_weather",
  "description": "查询指定城市的天气",
  "parameters": {
    "city": "string"
  }
}
```

用户：

```text
新加坡今天多少度？
```

模型不会直接“执行函数”，而可能生成类似：

```json
{
  "name": "get_weather",
  "arguments": {
    "city": "Singapore"
  }
}
```

然后：

```text
                 User
                   ↓
             "新加坡天气？"
                   ↓
        ┌──────────────────┐
        │       LLM        │
        │ 看到了 Tool Schema│
        └────────┬─────────┘
                 ↓
          选择 get_weather
                 ↓
         生成结构化参数
                 ↓
        ┌──────────────────┐
        │  Agent Runtime   │
        └────────┬─────────┘
                 ↓
        真正执行 get_weather()
                 ↓
             天气 API
                 ↓
              Result
                 ↓
               LLM
                 ↓
            自然语言回答
```

这里面试特别容易问一句：

> **到底是谁调用函数？**

一定要答：

> **LLM 负责“决定调用什么 + 生成参数”，真正执行函数的是宿主程序/Agent Runtime。**

---

# 二、模型怎么知道应该调用哪个 Tool？

不是模型自己扫描你的 Java/Python 代码。

而是应用把 Tool 的：

```text
Name
Description
Parameters Schema
```

提供给模型。

例如：
Function Calling 的原理是

```text
Tool 1:
name: query_database
description: 查询业务数据库

Tool 2:
name: search_logs
description: 查询服务运行日志

Tool 3:
name: restart_service
description: 重启指定服务
```

用户说：

```text
帮我查一下 payment-service 为什么报错
```

模型根据：

```text
用户意图
+
Tool Name
+
Tool Description
+
参数 Schema
+
当前 Context
```

判断：

```text
search_logs 更合适
```

所以 Tool Description 非常重要。

面试可以总结：

> **Tool Calling 本质上也是模型的一次结构化生成任务。工具描述质量、工具数量、工具之间的语义区分度都会影响工具选择准确率。**

---

# 三、Tool Use 和 Function Calling 什么关系？

可以理解为：

```text
Tool Use
  │
  ├── Function Calling
  ├── MCP Tool
  ├── Browser
  ├── Code Interpreter
  ├── Database
  └── Other Tools
```

**Tool Use 是更大的概念。**

Function Calling 是实现 Tool Use 的一种常见机制。

一句话：

> Tool Use 是“Agent 使用外部能力”这个概念，Function Calling 是实现这种能力的一种结构化接口机制。

---

# 四、Function Call 参数格式错了怎么办？

这是网页列出的阿里相关题。

比如模型输出：

```json
{
  "city": 123,
  "date": "abc"
}
```

但 Schema 要求：

```text
city: String
date: YYYY-MM-DD
```

生产环境不能直接执行。

一般：

```text
LLM
 ↓
Function Call
 ↓
Schema Validation
 ↓
是否合法？
 ↙       ↘
No       Yes
↓         ↓
Repair   Execute
↓
Retry
↓
Fallback
```

工程上通常做四层：

**第一层：Schema Validation**

JSON Schema / Pydantic / Java Bean Validation 检查：

```text
类型
必填字段
枚举
范围
格式
```

**第二层：自动修复**

简单错误可以：

```text
"123" → 123
字段默认值
日期格式转换
```

**第三层：让模型重新生成**

把错误反馈：

```text
city must be string
```

重新要求生成 Function Call。

**第四层：失败兜底**

达到最大重试次数：

```text
停止调用
↓
返回错误
↓
必要时人工处理
```

不要无限 Retry。

---

# 五、MCP 是什么？

这是这一章**最核心的问题之一**，网页连续列了多道 MCP 问题。

MCP：

> **Model Context Protocol**

可以把它理解成：

> **一套标准协议，用于让 AI 应用以统一方式连接外部工具、资源和上下文能力。**

以前：

```text
Agent
 ├── 自己接 GitHub
 ├── 自己接 Database
 ├── 自己接 Slack
 ├── 自己接 Filesystem
 └── 自己接 Jira
```

每个系统：

```text
API 不一样
认证不一样
Schema 不一样
接入方式不一样
```

MCP 希望变成：

```text
                  Agent / Host
                       │
                  MCP Client
                       │
               ─── MCP Protocol ───
                       │
              ┌────────┼────────┐
              ↓        ↓        ↓
          MCP Server MCP Server MCP Server
              ↓        ↓        ↓
           GitHub      DB      Files
```

核心价值：

> **标准化 Agent 和外部能力之间的连接方式，降低 M×N 的集成成本。**

---

# 六、MCP 解决了什么问题？

假设：

```text
10 个 Agent
×
20 种工具
```

没有统一协议，很容易变成大量定制 Adapter：

```text
Agent A → GitHub Adapter
Agent A → DB Adapter
Agent A → Slack Adapter

Agent B → GitHub Adapter
Agent B → DB Adapter
...
```

MCP 把连接方式标准化：

```text
Agent
  ↓
MCP Client
  ↓
统一协议
  ↓
MCP Server
  ↓
真实系统
```

所以面试可以回答：

> MCP 使AI应用发现、描述和调用外部工具的方式标准化。工具提供方通过 MCP Server 暴露工具名称、描述和参数 Schema，AI 应用通过 MCP Client 动态获取这些信息，因此不同 AI 应用不需要针对每个工具重复设计一套私有的接入协议。

这个区分很重要。

---

# 七、MCP 底层怎么工作？

网页明确问到了“底层原理”和“通信方式”。

理解成 Client-Server 即可：

```text
┌─────────────────────┐
│     MCP Host        │
│ Claude / IDE / Agent│
└──────────┬──────────┘
           │
      MCP Client
           │
     MCP Protocol
           │
      MCP Server
           │
   ┌───────┼─────────┐
   ↓       ↓         ↓
 Tools  Resources  Prompts
```

一个典型流程：

```text
① 建立连接
       ↓
② 初始化 / 能力协商
       ↓
③ 获取 Server 提供的能力
       ↓
④ Agent 选择 Tool
       ↓
⑤ MCP Client 发请求
       ↓
⑥ MCP Server 执行
       ↓
⑦ 返回 Result
       ↓
⑧ Result 进入 Agent Context
```

通信层面面试时可以说：

> MCP 是 Client-Server 架构，消息层采用 JSON-RPC 风格的协议；实际部署可以根据场景通过本地进程通信或基于 HTTP 的远程传输方式连接。

---

# 八、MCP Server 怎么开发？

这是网页中的原题。

假设做：

```text
Database MCP Server
```

首先定义能力：

```text
query_database
get_table_schema
list_tables
```

然后：

```text
        MCP Server
             │
     ┌───────┼─────────┐
     ↓       ↓         ↓
list_tables schema   query
                       ↓
                   Database
```

开发重点不是“套 SDK”这么简单，而是：

```text
① 定义工具
② 定义输入 Schema
③ 实现业务逻辑
④ 返回标准结果
⑤ 接入认证授权
⑥ 限流 / 超时 / 日志
⑦ 暴露给 MCP Client
```

面试最好补一句：

> 真正生产化的 MCP Server，重点往往不是协议实现，而是权限、凭证管理、参数校验、审计、限流和危险操作控制。

---

# 九、MCP 和传统 Tool 注册有什么区别？

传统 Function Tool：

```text
Agent Application
     │
     ├── registerTool(A)
     ├── registerTool(B)
     └── registerTool(C)
```

工具通常和应用绑定得比较紧。

MCP：

```text
Agent
 ↓
MCP Client
 ↓
MCP Server
 ↓
Tools
```

工具能力可以独立提供。

因此：

> **传统 Tool 注册偏应用内部集成；MCP 更强调跨应用、跨工具提供方的标准化能力发现和调用。**

---

# 十、MCP 和 Function Calling 到底有什么区别？

这是必背题，网页显示字节、快手、货拉拉、百度等都有相关问题。

不要回答：

> Function Calling 是旧技术，MCP 是新技术。

这是错误理解。

两者处于**不同层次**。

```text
                LLM
                 │
        Function Calling
                 │
       “我要调用 query_db”
                 ↓
              Runtime
                 │
             MCP Client
                 │
          MCP Protocol
                 │
             MCP Server
                 │
              Database
```

Function Calling 解决：

> **模型如何表达“我要调用哪个工具、参数是什么”。**

MCP 解决：

> **Agent 应用如何标准化发现、连接和使用外部能力。**

所以：

> **Function Calling 偏模型 ↔ Runtime 的工具选择与参数生成；MCP 偏 Runtime ↔ 外部工具提供方的标准化连接。两者不是替代关系，可以一起使用。**

这一句话建议直接记住。

---

# 十一、没有 MCP 之前怎么做？

网页直接问：

> 没有 MCP 之前，多 Agent 协同是怎么做的？

很简单：

```text
Function Calling
+
API
+
自定义 Adapter
+
消息队列/RPC
```

例如主 Agent：

```text
Function:
call_search_agent()
call_code_agent()
call_database_agent()
```

然后 Runtime 自己路由：

```text
Main Agent
   ↓ Function Call
Router
 ├→ Search Agent
 ├→ Code Agent
 └→ DB Agent
```

所以：

> MCP 不是 Agent 能调用工具的前提，它解决的是标准化问题。

---

# 十二、MCP 和 A2A 有什么关系？

网页也列出了这一题。

可以简单记：

```text
MCP
Agent ↔ Tool / Resource

A2A
Agent ↔ Agent
```

例如：

```text
                  Main Agent
                   /      \
                A2A        A2A
                 ↓          ↓
         Research Agent   Code Agent
                │            │
               MCP          MCP
                ↓            ↓
             Search        GitHub
```

一句话：

> **MCP 更偏 Agent 如何使用外部能力，A2A 更偏独立 Agent 之间如何发现、通信和协作。**

实际边界并不是绝对不能重叠，但这个回答面试非常清楚。

---

# 十三、Skills 是什么？

这是 2026 这一版题单特别强调的部分，网页连续列了 Skill、渐进式披露、Skill 系统设计和上百个 Skill 管理等问题。

Skill 可以理解为：

> **把某类任务的专业知识、执行流程、工具使用方法和约束封装成可复用的能力模块，让 Agent 在需要完成某类任务时加载并遵循这套方法。**

例如：

```text
PDF Skill
```

里面可能告诉 Agent：

```text
什么时候使用这个 Skill

怎么读取 PDF

怎么修改 PDF

应该调用什么工具

操作步骤是什么

修改后如何验证

失败怎么办
```

所以 Skill 不一定只是一个函数。

它更像：

```text
Skill
├── Instructions
├── Workflow
├── Domain Knowledge
├── Tool Usage
├── Constraints
├── Examples
└── Validation
```

这点非常重要。

---

# 十四、Skill 和 Function Call 有什么区别？

非常容易混。

Function：

```text
query_database(sql)
```

更像：

> **一个动作。**

Skill：

```text
Database Analysis Skill
```

可能规定：

```text
① 先获取 Schema
② 判断相关表
③ 生成只读 SQL
④ 校验 SQL
⑤ query_database
⑥ 分析 Result
⑦ 必要时再次查询
⑧ 输出结论
```

所以：

> **Function Call 是“执行一个动作”，Skill 是“完成一类任务的方法”。**

可以理解：

```text
Skill
 │
 ├── Step
 ├── Step
 ├── Function Call
 ├── Function Call
 └── Validation
```

---

# 十五、MCP 和 Skills 有什么区别？

这是网页这一章里公司覆盖最多的问题之一。

建议你直接记：

> **MCP 管“有什么工具可以用”，Skill 管“这个任务应该怎么做”。**

例如：

```text
MCP Server
提供：
    query_logs
    query_metrics
    query_database
    restart_service
```

而：

```text
Incident Diagnosis Skill

告诉 Agent：

1. 先查 metrics
2. 再查 logs
3. 判断 DB 是否异常
4. 必要时 query_database
5. 不允许直接 restart
6. 输出 Root Cause
```

所以：

```text
MCP = Capability / Connectivity

Skill = Knowledge / Procedure
```

---

# 十六、Prompt、MCP、Skill、Subagent 怎么区分？

网页也直接列了这道字节相关题。

这个表建议记住：

| 概念 | 解决什么问题 | 可以理解为 |
|---|---|---|
| Prompt | Agent 应该遵循什么基本指令 | 规则 |
| Function Call | 执行哪个具体动作 | 函数调用 |
| MCP | 外部能力怎么标准化接入 | 接口协议 |
| Skill | 某类任务应该怎么完成 | SOP / 能力包 |
| Subagent | 谁负责完成某个子任务 | 专业员工 |

比如：

```text
主 Agent
│
├── Prompt
│   "你是运维 Agent"
│
├── Skill
│   "故障排查 SOP"
│
├── MCP
│   ├── Logs
│   ├── Metrics
│   └── Database
│
└── Subagent
    └── Database Agent
```

这张关系图非常适合面试现场画。

---

# 十七、模型怎么知道应该调用哪个 Skill？

和 Tool 有点类似，但通常采用：

```text
Skill Metadata
+
Description
+
当前任务
```

例如只给模型：

```text
Skill:
incident-diagnosis

Description:
用于分析线上服务异常、接口超时、
错误率升高等生产故障。
```

用户：

```text
payment API P99 突然从 100ms 涨到 4s
```

模型匹配：

```text
incident-diagnosis
```

然后才加载 Skill 的详细内容。

这里就引出了下一道高频题。

---

# 十八、为什么 Skill 能减少 Token？

网页明确问到了这道题。

假设系统有：

```text
100 个 Skills
```

每个 Skill 详细说明：

```text
3000 tokens
```

如果全部塞进 Prompt：

```text
300,000 tokens
```

非常浪费。

Skill 可以采用：

> **Progressive Disclosure（渐进式披露）**

一开始只给：

```text
Skill Name
+
Description
+
少量 Metadata
```

例如：

```text
pdf-processing
处理 PDF

database-analysis
数据库分析

incident-diagnosis
线上故障排查
```

模型选择：

```text
incident-diagnosis
```

再加载：

```text
incident-diagnosis/skill.md
```

所以：

```text
100 Skills
       ↓
只加载 Metadata
       ↓
模型选择 Skill 37
       ↓
加载 Skill 37 详细 Instructions
       ↓
必要时再加载 Reference / Script
```

这就是：

> **需要什么，什么时候再加载什么。**

从而降低 Context 占用。

---

# 十九、什么是 Skill 的渐进式披露？

这道建议直接用“三层”回答。

```text
Level 1
Metadata
name + description
      ↓
模型判断是否相关

Level 2
Skill Instructions
详细工作流程
      ↓
执行任务

Level 3
Resources
reference / script / template
      ↓
需要时再加载
```

目的不只是省 Token。

还有：

```text
减少上下文污染
减少无关指令干扰
提升 Skill 选择准确率
允许系统维护大量 Skills
```

所以更完整的回答是：

> 渐进式披露本质上是一种 Context Engineering：不是把所有能力说明一次性塞给模型，而是根据当前任务逐层暴露最相关的信息。

---

# 二十、Skill 和 Rule 有什么区别？

网页显示腾讯、蚂蚁相关面试问过。

可以这样理解：

### Rule

强调：

> **必须遵守什么。**

例如：

```text
禁止删除生产数据库
所有 SQL 必须只读
禁止输出用户密码
```

通常是：

```text
Always On
```

### Skill

强调：

> **某类任务应该怎么完成。**

例如：

```text
SQL Analysis Skill

1. 查看 Schema
2. 找相关表
3. 生成 SQL
4. 校验
5. 执行
6. 分析
```

所以：

```text
Rule  = Constraint
Skill = Capability / Procedure
```

一句话：

> **Rule 管“不能做什么、必须遵守什么”，Skill 管“这个事情应该怎么做”。**

---

# 二十一、如果让你设计一个 Skill 系统，怎么设计？

这是网页中的快手相关题。

我会拆成：

```text
                 Skill Registry
                      │
        ┌─────────────┼─────────────┐
        ↓             ↓             ↓
      Search        Version      Permission
        ↓
    Skill Selector
        ↓
    Skill Loader
        ↓
      Runtime
        ↓
     Tool / MCP
        ↓
     Evaluation
```

一个 Skill 至少应该有：

```yaml
name:
description:
version:
trigger:
instructions:
tools:
permissions:
constraints:
examples:
references:
validation:
```

例如：

```yaml
name: incident-diagnosis

description:
  Diagnose production service incidents.

tools:
  - query_logs
  - query_metrics

permissions:
  - read_logs
  - read_metrics

constraints:
  - never restart production automatically
```

面试再补：

> Skill 需要版本管理、权限控制、评测和可观测性，不能只是把一堆 Prompt 文件放进目录。

---

# 二十二、100 个甚至 1000 个 Skill 怎么管理？

网页专门问：

> 注册了上百个候选 Skill，该怎么处理——检索、分层还是渐进式披露？

肯定不能：

```text
1000 Skills
↓
全部塞给 LLM
```

通常：

```text
                User Query
                    ↓
             Skill Retrieval
                    ↓
                 Top-K
                    ↓
              Skill Router
                    ↓
              3~5 Skills
                    ↓
                 LLM
                    ↓
              Selected Skill
                    ↓
             Load Full Skill
```

检索可以结合：

```text
Keyword
Embedding
Metadata
Category
Permission
历史成功率
```

也可以先分层：

```text
             1000 Skills
                  ↓
              Category
        ┌─────────┼─────────┐
        ↓         ↓         ↓
      Coding     Data      Office
        ↓
      Debug
        ↓
     Top-K Skill
```

核心思想：

> **先缩小候选空间，再让模型选择，最后加载完整 Skill。**

---

# 二十三、100 个 Tool 怎么避免模型选错？

这也是网页的高频题。

和 Skill 类似：

```text
不要：
100 Tools → 全塞 LLM

而是：

User
 ↓
Intent / Router
 ↓
Tool Retrieval
 ↓
Top-K Tools
 ↓
LLM Selection
 ↓
Schema Validation
 ↓
Execute
```

具体手段：

```text
① Tool Description 写清楚
② 减少语义重叠
③ Tool 分类
④ Tool Retrieval
⑤ 权限过滤
⑥ 参数 Schema 严格化
⑦ Tool Choice Evaluation
⑧ 调用结果验证
```

特别是：

```text
get_user
query_user
find_user
search_user
lookup_user
```

这种大量语义近似工具，非常容易导致 Tool Selection 混乱。

---

# 二十四、工具编排应该放 Skill 还是 Agent？

网页也直接问到了这个设计问题。

没有绝对答案。

如果流程是领域固定 SOP：

```text
查询指标
↓
查询日志
↓
查询 DB
```

适合写进：

```text
Skill
```

如果：

```text
下一步高度依赖实时 Observation
```

应该由：

```text
Agent
```

动态决定。

所以：

> **稳定、可复用的领域流程沉淀到 Skill；需要根据环境动态决策的控制逻辑留给 Agent。**

这是非常好的架构原则。

---

# 二十五、所有 Agent 都应该访问所有 MCP 吗？

**绝对不应该。**

例如：

```text
Research Agent

允许：
Web Search
Read DB

禁止：
Delete DB
Restart Server
Transfer Money
```

采用：

> **Least Privilege Principle——最小权限原则。**

架构：

```text
             Agent Identity
                   ↓
              Permission
                   ↓
               MCP Router
            ↙       ↓       ↘
         Allow    Allow     Deny
          Logs      DB     Restart
```

权限最好同时考虑：

```text
Agent
User
Tool
Resource
Environment
Action
```

例如：

```text
开发环境 → 可以 restart
生产环境 → 必须人工审批
```

---

# 二十六、工具超时、报错、连续失败怎么处理？

网页把这一项也列为工程面试题。

生产 Agent 必须有：

```text
Tool Call
   ↓
Timeout
   ↓
Retry
   ↓
Backoff
   ↓
Circuit Breaker
   ↓
Fallback
```

例如：

```text
第一次失败
↓
等待
↓
Retry 1
↓
Retry 2
↓
仍然失败
↓
Circuit Breaker
↓
换备用工具 / 返回部分结果
```

同时区分：

```text
参数错误
→ 不应该盲目 Retry

网络超时
→ 可以 Retry

权限错误
→ Retry 没意义

服务不可用
→ Circuit Breaker
```

这句话非常重要：

> **Retry 应该根据错误类型决定，而不是所有失败都重试。**

---

# 二十七、多工具调用系统怎么设计？

网页对应百度相关题。

一个比较完整的架构：

```text
                   User
                     ↓
                  Agent
                     ↓
                Tool Router
                     ↓
               Tool Retrieval
                     ↓
               LLM Selection
                     ↓
             Schema Validation
                     ↓
               Tool Executor
            ┌────────┼────────┐
            ↓        ↓        ↓
           MCP      API      Local
            ↓        ↓        ↓
            └────────┼────────┘
                     ↓
                  Result
                     ↓
              Result Validator
                     ↓
                  Agent
```

外围再加：

```text
Permission
Timeout
Retry
Circuit Breaker
Tracing
Metrics
Audit
```

这才比较接近生产级 Tool Runtime。

---

# 二十八、Function Call、MCP、Skill 三者到底怎么记？

这是我最建议你真正理解的一张图，也是网页明确列出的 B 站相关题。

假设用户说：

> **帮我排查 payment-service 为什么响应变慢。**

整个过程：

```text
                       User
                        │
              "排查 payment 慢"
                        ↓
                ┌──────────────┐
                │    Agent     │
                └──────┬───────┘
                       │
                  选择 Skill
                       ↓
            ┌─────────────────────┐
            │ Incident Diagnosis  │
            │        Skill        │
            │                     │
            │ 1. 查 Metrics       │
            │ 2. 查 Logs          │
            │ 3. 查 DB            │
            └──────────┬──────────┘
                       ↓
              Function Calling
                       │
              query_metrics(...)
                       ↓
                  MCP Client
                       ↓
                 MCP Server
                       ↓
                Metrics System
```

分别回答：

```text
Skill
↓
告诉 Agent
“这类任务怎么做”

Function Calling
↓
告诉 Runtime
“我要执行哪个动作、参数是什么”

MCP
↓
告诉系统
“怎么标准化连接外部能力”
```

所以我建议你直接记这一句：

> **Skill 管方法，Function Call 管动作，MCP 管连接。**

---

# 二十九、这一章面试真正应该背什么？

网页这一节从 Function Calling 一直列到 Spring AI Alibaba 的 Function Calling 实现，而且备考建议明确指出，工具调用（Function Calling、MCP、Skills）是当前 Agent 应用开发岗位的重点之一。

你不需要平均背 29 道题。优先把下面 **8 个问题**练到能直接说：

1. **Function Calling 的完整执行流程是什么？谁真正执行 Function？**
2. **模型为什么知道该调用哪个 Tool？**
3. **MCP 是什么，解决什么问题？**
4. **MCP 和 Function Calling 有什么区别？**
5. **Skill 是什么？和 Function Calling 有什么区别？**
6. **MCP 和 Skill 有什么区别？**
7. **为什么 Skill 要做渐进式披露？**
8. **如果有 100 个 Tool / 100 个 Skill，怎么让 Agent 准确选择？**

最后把整个知识体系压成：

```text
                         Agent
                           │
              ┌────────────┼────────────┐
              ↓            ↓            ↓
            Prompt        Skill      Subagent
              │            │
           基础规则       任务方法      专业角色
                           │
                     Function Call
                           │
                        具体动作
                           │
                      MCP Client
                           │
                    标准化连接协议
                           │
                      MCP Server
                           │
               ┌───────────┼───────────┐
               ↓           ↓           ↓
              DB          Logs        GitHub
```

**面试口诀：**

> **Prompt 定规则，Skill 教方法，Function Call 做动作，MCP 接能力，Subagent 做分工。**

你把这句话和上面那张图真正理解之后，这个网页「工具调用：Function Calling、MCP 与 Skills」这一整章的大多数问题就不是独立的八股文了，而是可以从同一个架构体系里推导出来。