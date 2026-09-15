我重新对照了你给的题单。「Agent Harness 与运行时工程」一共 **13 道题**，从 Prompt/Context/Harness Engineering，一直到框架选型、Runtime、LangGraph、Spring AI 和失败降级。网页也特别指出 Harness 是 2026 年新增的重点方向，考察重点是**理解设计动机，而不是背名词**。

下面给你一份适合直接保存复习的 Markdown 版。

# Agent Harness 与运行时工程面试题总结

## 0. 先理解：什么是 Agent Harness？

**Agent Harness = 包裹在模型外部、负责让 Agent 稳定完成任务的一整套运行与控制机制。**

LLM 只是负责：

```text
理解 → 推理 → 决策
```

但真正的 Agent 还需要：

```text
                Agent Harness
┌─────────────────────────────────────┐
│                                     │
│   Context Management                │
│   Tool Management                   │
│   Agent Loop                        │
│   State Management                  │
│   Memory                            │
│   Permission / Guardrail            │
│   Retry / Timeout                   │
│   Subagent                          │
│   Observability                     │
│                                     │
│              ┌─────┐                │
│              │ LLM │                │
│              └─────┘                │
└─────────────────────────────────────┘
```

一句话：

> **模型决定“想怎么做”，Harness 负责“让它真正稳定、安全、可控地做完”。**

---

# 1. Prompt Engineering、Context Engineering、Harness Engineering 有什么区别？

### Prompt Engineering

解决：

> **怎么给模型下好指令？**

关注：

- System Prompt
- 指令结构
- Few-shot
- 输出格式
- 角色定义

```text
重点：把一句话“写好”
```

---

### Context Engineering

解决：

> **这一轮到底应该让模型看到什么？**

关注：

- 对话历史
- RAG
- Memory
- Tool Result
- Context Compression
- 信息优先级

```text
重点：把正确的信息“放进去”
```

---

### Harness Engineering

解决：

> **怎么控制模型在一个长期、多步骤任务中稳定运行？**

关注：

- Agent Loop
- Tool 调度
- State
- Context 生命周期
- Retry / Timeout
- 权限
- Subagent
- Checkpoint
- Observability

### 一句话区分

```text
Prompt Engineering
= 怎么告诉模型做事

Context Engineering
= 给模型哪些信息

Harness Engineering
= 怎么让模型持续、可靠地把事情做完
```

---

# 2. Agent Engineering 和 Harness 分别是什么？

### Agent Engineering

是更大的概念：

> **设计、开发、评测和部署完整 Agent 系统的工程实践。**

包括：

```text
Agent Engineering
├── Model
├── Prompt
├── Context
├── Tools
├── MCP
├── Skills
├── Memory
├── Harness
├── Evaluation
└── Deployment
```

### Harness

是 Agent Engineering 中非常重要的一层：

```text
Agent Engineering
        │
        ├── Model
        ├── Tools
        ├── Skills
        │
        └── Harness
              ↓
         控制 Agent 怎么跑
```

**标准回答：**

> Agent Engineering 是完整 Agent 系统的工程化过程，而 Harness 更关注模型外围的执行控制层，包括上下文、工具、状态、循环、权限、异常恢复和可观测性等。

---

# 3. LangChain 早就在做这些，为什么现在 Harness 突然火了？

因为以前 Agent 通常比较简单：

```text
Prompt
 ↓
LLM
 ↓
Tool
 ↓
Answer
```

LangChain 主要解决：

> **组件怎么连接、Tool 怎么调用、Chain 怎么组织。**

但现在 Agent 开始执行：

```text
几十分钟甚至几小时任务
+
大量 Tool Call
+
动态 Context
+
文件修改
+
Subagent
+
任务恢复
+
权限控制
```

问题变成：

> **不是“模型会不会调用工具”，而是“模型怎么长期稳定地运行”。**

因此 Harness 强调的是：

```text
Context Lifecycle
State
Checkpoint
Recovery
Tool Governance
Permission
Observability
Long-running Loop
```

### 面试总结

> LangChain 等框架早就包含部分 Harness 能力，但 Harness 这个概念现在被单独强调，是因为 Agent 从简单的 LLM Chain 发展成了长期运行、可执行真实操作的系统，工程重点从“连接组件”转向“管理 Agent 的整个执行生命周期”。

---

# 4. Harness、Hermes 等新 Agent 范式怎么理解？

这类设计的共同趋势是：

> **把模型本身和模型外部的执行环境分开。**

过去：

```text
更强 Agent
≈
更强模型
```

现在越来越强调：

```text
Agent 能力
=
Model
+
Harness
+
Tools
+
Context
+
Memory
+
Environment
```

也就是说：

> 同一个模型，在不同 Harness 下，最终完成复杂任务的能力可能差异很大。

面试重点不要死背某个项目实现，而要说清趋势：

**Agent 的能力不只来自模型权重，也越来越来自外围运行系统的设计。**

---

# 5. 主流 Agent Harness / Agent 框架有哪些？

可以按层次回答：

### 通用 Agent 编排

```text
LangGraph
LangChain
AutoGen
CrewAI
Semantic Kernel
```

### Coding / Autonomous Agent

```text
OpenHands
Claude Code 类 Coding Harness
SWE-agent
```

### Java 生态

```text
Spring AI
Spring AI Alibaba
LangChain4j
```

不要只背框架名称。

面试官真正想听：

> **你为什么选这个框架？**

选择维度：

```text
状态管理
工作流控制
Tool / MCP 支持
Checkpoint
多 Agent
可观测性
生态
稳定性
团队技术栈
```

---

# 6. Claude Code 的上下文管理怎么理解？

核心问题：

> Coding Agent 不可能永远把整个代码库和全部历史对话塞进 Context。

因此一般需要：

```text
用户任务
   ↓
相关文件发现
   ↓
按需读取代码
   ↓
Tool Result
   ↓
Working Context
   ↓
上下文越来越大
   ↓
压缩 / 摘要 / 卸载
   ↓
继续执行
```

核心思想：

### ① 按需加载

不是：

```text
整个 Repo → Context
```

而是：

```text
先搜索
→ 找相关文件
→ 再读取
```

### ② Tool Result 控制

大量日志、代码、搜索结果不能无限进入 Context。

### ③ Context Compression

历史过长时：

```text
旧历史
↓
Summary
↓
保留关键状态
```

### ④ 外部状态保存

文件、执行结果等不一定长期放在 Context：

```text
Context
≈ Working Memory

Filesystem / State Store
≈ External Memory
```

一句话：

> **核心是让 Context 只保存当前决策真正需要的信息，而不是把所有历史永久塞给模型。**

---

# 7. 为什么选 LangGraph，而不是 Deep Agents？不稳定可能来自哪里？

框架选型不要回答：

> A 比 B 强。

应该看业务。

如果系统要求：

```text
流程可控
状态明确
需要 Checkpoint
需要 HITL
需要失败恢复
需要确定性节点
```

更适合 Graph / State Machine 思路：

```text
LangGraph
```

如果追求：

```text
高度自主
动态规划
动态 Subagent
开放任务
```

更 Autonomous 的 Agent 框架可能更方便，但也更容易出现：

```text
路径不可预测
上下文膨胀
Tool 调用失控
错误传播
调试困难
结果不稳定
```

### 标准回答

> 我的选型标准不是哪个框架更“智能”，而是业务需要多少自主性和多少确定性。生产业务如果强调状态可控、可恢复和可观测，我会更倾向显式 Graph；开放式复杂任务才考虑更高自主性的 Deep Agent。

---

# 8. Agent Runtime 越来越多，平台怎么统一适配？

这是很重要的**平台架构题**。

不要让业务直接依赖：

```text
LangGraph API
Claude API
OpenAI Agent API
框架 A API
框架 B API
```

中间增加统一抽象层：

```text
                 Agent Platform
                       │
              Runtime Interface
                       │
       ┌───────────────┼──────────────┐
       ↓               ↓              ↓
 LangGraph Adapter   Runtime B      Runtime C
       ↓               ↓              ↓
 LangGraph          Framework B    Framework C
```

统一抽象：

```text
run()
resume()
cancel()
getState()
toolCall()
checkpoint()
stream()
```

同时统一：

```text
Message Schema
Tool Schema
State Schema
Tracing
Metrics
Error Model
Permission
```

### 核心思想

> **通过 Runtime Interface + Adapter 隔离底层框架差异，让业务层不直接绑定某一个 Agent Runtime。**

---

# 9. Super Agent 还是 Host Agent + Sub Agents？

一般不建议：

```text
一个超级 Agent
+
几百个 Tools
+
什么都负责
```

容易出现：

```text
Context 过大
Tool Selection 准确率下降
权限过大
Prompt 冲突
调试困难
```

复杂系统更适合：

```text
                 Host Agent
                     │
        ┌────────────┼────────────┐
        ↓            ↓            ↓
     Code Agent   Search Agent   DB Agent
        │            │            │
      Tools        Tools        Tools
```

Host Agent：

```text
理解目标
任务拆分
路由
结果汇总
```

Sub Agent：

```text
处理垂直领域任务
拥有有限 Tool
拥有独立 Context
拥有最小权限
```

### 标准回答

> 通用入口可以是 Host Agent，但执行能力更适合拆成专业 Sub Agent。这样能够降低上下文和工具选择复杂度，同时实现权限隔离和独立评测。

---

# 10. LangChain 和 LangGraph 有什么区别？

### LangChain

更偏：

> **LLM 应用组件和调用链的抽象。**

例如：

```text
Prompt
↓
LLM
↓
Retriever
↓
Tool
↓
Parser
```

适合相对直接的 Chain / Agent 组装。

### LangGraph

更偏：

> **有状态的 Agent Workflow / Graph 编排。**

例如：

```text
        Planner
           ↓
        Executor
        ↙      ↘
     Success   Fail
       ↓         ↓
     Final      Retry
                 ↓
              Planner
```

LangGraph 更强调：

```text
State
Node
Edge
Conditional Edge
Checkpoint
Resume
Human-in-the-loop
```

### 一句话

```text
LangChain
≈ 组件与调用链

LangGraph
≈ 有状态 Agent 工作流
```

---

# 11. LangGraph 的状态快照 / Checkpoint 怎么实现？

核心思想：

> **每执行到关键节点，就把当前 Graph State 持久化。**

例如：

```text
State = {
    messages,
    current_plan,
    tool_results,
    current_step,
    metadata
}
```

执行：

```text
Node A
 ↓
State V1 → Checkpoint

Node B
 ↓
State V2 → Checkpoint

Node C
 ↓
程序崩溃
```

恢复时：

```text
读取 State V2
↓
恢复 Graph
↓
继续执行 Node C
```

Checkpoint 可以存：

```text
Memory
Database
Redis
Persistent Store
```

它主要解决：

```text
断点恢复
长任务
HITL 暂停
失败重试
状态回放
```

---

# 12. Java 为什么选 Spring AI 而不是 LangChain4j？

这题**不要说 Spring AI 一定更好**。

应该从项目背景回答。

如果项目本身是：

```text
Spring Boot
Spring Cloud
企业 Java 后端
```

Spring AI 的优势通常在于：

```text
Spring 原生集成
Dependency Injection
Configuration
Observability
Model / Vector Store 抽象
Tool Calling
企业已有基础设施兼容
```

因此：

> 如果团队已有成熟 Spring 技术栈，我会优先考虑 Spring AI，因为接入现有配置、Bean 生命周期、监控和企业基础设施的成本更低。

LangChain4j 的优势则可以是：

```text
Java-first API
Agent / AI 抽象直接
开发体验简洁
独立于 Spring 也容易使用
```

### 面试关键

> **框架选型看团队技术栈、稳定性、生态、功能覆盖和维护成本，不是看谁名字更火。**

---

# 13. 非 Java 生态有哪些主流 Agent 框架？

Python 生态常见：

```text
LangChain
LangGraph
AutoGen
CrewAI
Semantic Kernel
OpenAI Agents SDK
```

以及更垂直的：

```text
OpenHands
SWE-agent
```

面试不要只列名字。

可以补一句：

> LangGraph 更适合显式状态机和复杂 Workflow；AutoGen / CrewAI 更偏多 Agent 协作；OpenHands / SWE-agent 更偏 Coding Agent 场景。

---

# 14. Agent 的组装链 / 执行链怎么设计？失败如何降级？

一个生产 Agent 的执行链可以设计为：

```text
User Request
     ↓
Intent / Router
     ↓
Context Builder
     ↓
Planner
     ↓
Agent / LLM
     ↓
Tool Selection
     ↓
Permission Check
     ↓
Tool Execution
     ↓
Result Validation
     ↓
State Update
     ↓
Finish?
 ↙           ↘
No            Yes
↓              ↓
Loop         Response
```

## 失败处理

不要只写：

```text
失败 → Retry
```

应该分层：

### 第一层：局部重试

```text
网络超时
临时 API 失败

→ Retry + Backoff
```

### 第二层：工具降级

```text
Tool A 不可用
↓
Tool B / Cache / Backup API
```

### 第三层：模型降级

```text
主模型不可用
↓
备用模型
```

### 第四层：策略降级

```text
Agent 无法自主完成
↓
退化成 Workflow / Search / RAG
```

### 第五层：人工介入

```text
高风险
持续失败
结果置信度低
↓
HITL
```

核心原则：

> **Agent 的失败不能只有“成功 / 崩溃”两种状态，而应该设计可恢复、可降级、可人工接管的执行链。**

---

# 最后：Harness 和 Runtime 到底什么关系？

这两个词最容易混。

可以粗略理解：

```text
              Agent Harness
                    │
        “Agent 应该怎么被控制”
                    │
      ┌─────────────┼─────────────┐
      ↓             ↓             ↓
 Context         Tool Policy     Retry
 Memory          Permission      Checkpoint
 Loop            Guardrail       Subagent
                    │
                    ↓
              Agent Runtime
                    │
          “真正把这些机制跑起来”
                    ↓
                  LLM
                    ↓
                 Tools
```

因此：

> **Harness 更偏整个外围控制体系和设计；Runtime 更偏真正承载 Agent 执行的运行时系统。**

实际工程里两者边界并不是绝对的，很多 Runtime 本身就实现了 Harness 的大量能力。

---

# 面试速记版

## 三种 Engineering

```text
Prompt Engineering
→ 怎么说

Context Engineering
→ 给模型看什么

Harness Engineering
→ 怎么让 Agent 稳定跑
```

## Harness 核心

```text
Harness
=
Context
+ State
+ Tools
+ Loop
+ Memory
+ Permission
+ Retry
+ Checkpoint
+ Observability
```

## Runtime

```text
Runtime
=
真正执行 Agent 的运行环境
```

## 框架

```text
LangChain
→ 组件 / Chain

LangGraph
→ State + Graph + Checkpoint

Spring AI
→ Spring 生态 AI 应用框架
```

## 架构趋势

```text
不是：

更强 Agent = 更强 LLM

而是：

Agent 能力
=
Model
+ Harness
+ Context
+ Tools
+ Skills
+ Runtime
```

## 最值得背的一句话

> **LLM 提供智能，Tool 提供能力，Context 提供信息，Skill 提供方法，而 Harness / Runtime 负责把这些东西组织起来，让 Agent 能够长期、稳定、安全、可恢复地完成任务。**

以上覆盖了网页「Agent Harness 与运行时工程」标题下列出的全部题目；题单尤其强调 Harness、Skill、上下文工程是 2026 年新出现的重点，并且实际面试会继续追问“项目里具体怎么做”。