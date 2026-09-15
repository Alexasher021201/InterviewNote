可以。你准备的是 **Agent 开发岗**，Transformer 不需要卷到算法岗那种“手推所有公式”的程度，但一定要理解 **Token → Embedding → Attention → Transformer → LLM → Agent** 这条链，以及它为什么会影响 Context、RAG、Tool Calling、长上下文和推理成本。

我按「基础概念 → 工作流程 → Agent 面试高频问题」给你梳理。

# 一、Transformer 是什么？

一句话：

> **Transformer 是一种以 Attention（注意力机制）为核心的神经网络架构，用于建模序列中不同 Token 之间的关系。**

2017 年论文 *Attention Is All You Need* 提出了 Transformer。

现在很多 LLM 的基础架构都来源于 Transformer。

关系可以先理解为：

```text
Transformer
    ↓
大规模训练
    ↓
Large Language Model
    ↓
LLM + Tools + Memory + Runtime...
    ↓
Agent
```

所以：

> **Transformer 是底层模型架构，LLM 是基于它训练出来的模型，而 Agent 是围绕 LLM 构建的应用系统。**

---

# 二、为什么 Transformer 很重要？

假设一句话：

```text
小明把苹果给了小红，因为她很饿。
```

理解「她」是谁，需要建立 Token 之间的关系：

```text
她 ←→ 小红
她 ←→ 很饿
苹果 ←→ 给
小明 ←→ 给
```

Transformer 的核心能力就是：

> **让每个 Token 根据上下文关注其他相关 Token，从而得到包含上下文语义的表示。**

这就是 Attention 最核心的直觉。

---

# 三、Transformer 整体流程

先看最重要的一张图：

```text
原始文本
   ↓
Tokenizer
   ↓
Tokens
   ↓
Token Embedding
   +
Position Information
   ↓
┌────────────────────────┐
│   Transformer Block    │
│                        │
│  Self-Attention        │
│        ↓               │
│  Feed Forward Network  │
│        ↓               │
│  Residual + Norm       │
└────────────────────────┘
   ↓
重复 N 层
   ↓
Hidden States
   ↓
Linear + Softmax
   ↓
预测下一个 Token
```

下面把这些东西拆开。

---

# 四、Tokenization 是什么？

LLM 通常不是直接按“一个汉字/一个英文单词”处理文本，而是先通过 Tokenizer 切成 Token。

例如：

```text
I love artificial intelligence
```

可能被拆成：

```text
["I", " love", " artificial", " intelligence"]
```

复杂单词也可能拆成多个 Token。

然后每个 Token 被转换成整数 ID：

```text
"I"            → 42
"love"         → 5832
"artificial"   → 9137
```

所以：

> **Token 是模型处理文本的基本单位。**

这和 Agent 开发直接相关，因为：

```text
Context Window
Token Cost
Prompt 长度
RAG Chunk
Tool Result
Memory
```

最终都受到 Token 数量影响。

---

# 五、Embedding 是什么？

模型不能直接理解：

```text
"apple"
```

所以需要把 Token 转换成一个向量：

```text
apple
 ↓
Token ID
 ↓
Embedding
 ↓
[0.12, -0.53, 0.81, ..., 0.27]
```

这个高维向量就是：

> **Embedding：用向量表示 Token。**

语义相近的内容，在某些表示空间里往往也具有相近关系。

例如：

```text
cat
dog

相对接近

cat
database

相对远
```

### Agent 面试注意

这里很容易继续问：

> Transformer 的 Token Embedding 和 RAG 的 Embedding 是不是一个东西？

概念相关，但用途不同。

**Token Embedding：**

```text
Token → 向量
↓
给 Transformer 内部处理
```

**RAG Embedding：**

```text
Document / Query
↓
Embedding Model
↓
语义向量
↓
Vector Database
↓
Similarity Search
```

不要直接把两者说成一回事。

---

# 六、为什么需要 Position Information？

Attention 本身并天然理解：

```text
A 在 B 前面
```

还是：

```text
B 在 A 前面
```

所以模型需要位置信息。

例如：

```text
我 喜欢 你
0   1   2
```

与：

```text
你 喜欢 我
0   1   2
```

Token 类似，但顺序完全改变了语义。

因此需要把：

```text
Token 信息
+
Position 信息
```

一起提供给模型。

经典 Transformer 使用 Positional Encoding；现代 LLM 也常使用 RoPE 等位置编码方案。

面试回答到这里通常足够。

---

# 七、Attention 到底是什么？

这是 Transformer **最重要的知识点**。

一句话：

> **Attention 让当前 Token 根据相关程度，有选择地聚合其他 Token 的信息。**

例如：

```text
The animal didn't cross the street because it was tired.
```

模型处理：

```text
it
```

时应该重点关注：

```text
animal
```

而不是：

```text
street
```

可以抽象成：

```text
             it
              │
       Attention Score
              │
     ┌────────┼────────┐
     ↓        ↓        ↓
  animal    street    tired
   0.7       0.1      0.2
```

最终 `it` 的表示会更多吸收与它相关的信息。

---

# 八、Q、K、V 是什么？

这是 Transformer 面试**非常容易问**的。

Attention 中每个 Token 会通过不同矩阵映射得到：

```text
Q = Query
K = Key
V = Value
```

可以用“搜索”来理解。

假设当前 Token 在问：

> **我应该关注谁？**

它的：

```text
Query
```

就是自己的查询需求。

其他 Token 提供：

```text
Key
```

用于判断：

> 我和你相关吗？

如果相关，再取它的：

```text
Value
```

也就是：

> 你真正提供给我的信息。

所以：

```text
Query
  ↓
和所有 Key 比较
  ↓
得到 Attention Score
  ↓
Softmax
  ↓
Attention Weight
  ↓
对 Value 加权求和
```

经典公式：

\[
Attention(Q,K,V)=Softmax(\frac{QK^T}{\sqrt{d_k}})V
\]

Agent 开发面试一般不需要推导，但最好知道每部分含义：

```text
QKᵀ
→ 计算相关性

÷ √dk
→ 控制数值尺度，避免点积过大

Softmax
→ 转成注意力权重

× V
→ 根据权重聚合信息
```

---

# 九、Self-Attention 是什么？

如果：

```text
Q、K、V
```

都来自**同一个输入序列**，就是 Self-Attention。

例如：

```text
小明 把 苹果 给了 小红
```

每个 Token 都可以关注这个序列里的其他 Token：

```text
小明 ─────→ 给了
苹果 ─────→ 给了
小红 ─────→ 给了
```

所以：

> **Self-Attention 用来建立同一序列内部 Token 之间的依赖关系。**

---

# 十、Multi-Head Attention 又是什么？

一个 Attention Head 可能主要学习一种关系。

所以 Transformer 使用多个 Head：

```text
                Tokens
                  ↓
       ┌──────────┼──────────┐
       ↓          ↓          ↓
     Head 1     Head 2     Head 3
       ↓          ↓          ↓
   语法关系     指代关系     语义关系
       └──────────┼──────────┘
                  ↓
                Concat
                  ↓
              Linear
```

注意这里的“语法/指代”只是帮助理解，并不是规定某个 Head 必须负责某一种功能。

标准回答：

> **Multi-Head Attention 让模型在不同表示子空间中并行学习不同类型的 Token 关系，提高模型表达能力。**

---

# 十一、Feed Forward Network 是干什么的？

Attention 完成：

> **Token 之间的信息交换。**

FFN 则进一步：

> **对每个 Token 的表示进行非线性变换和特征提取。**

可以粗略理解：

```text
Self-Attention
↓
“从别人那里获取相关信息”

FFN
↓
“进一步加工自己的表示”
```

一个 Transformer Block 大体就是：

```text
Input
 ↓
Self-Attention
 ↓
Add & Norm
 ↓
Feed Forward
 ↓
Add & Norm
 ↓
Output
```

然后重复很多层。

---

# 十二、Residual Connection 和 LayerNorm

### Residual Connection

类似：

```text
Output = F(x) + x
```

也就是保留原始信息的同时学习新的变化。

主要帮助深层网络：

```text
稳定训练
改善梯度传播
```

### LayerNorm

主要作用：

> 稳定神经网络中间表示和训练过程。

Agent 开发面试通常知道作用即可，不需要深入数学推导。

---

# 十三、Encoder 和 Decoder 是什么？

原始 Transformer 有：

```text
Encoder
+
Decoder
```

### Encoder

偏向：

> **理解输入。**

例如经典 BERT 是 Encoder-only 架构。

```text
Input
 ↓
Encoder
 ↓
Context Representation
```

适合：

```text
分类
文本理解
信息抽取
```

### Decoder

偏向：

> **根据已有 Token 自回归地预测下一个 Token。**

GPT 系列属于典型的 Decoder-only Transformer 路线。

```text
Prompt
 ↓
Decoder
 ↓
Token 1
 ↓
Token 2
 ↓
Token 3
 ↓
...
```

很多现代 LLM 使用 Decoder-only 架构。

---

# 十四、什么是 Causal / Masked Self-Attention？

生成文本的时候不能“偷看未来答案”。

例如：

```text
I love AI
```

预测：

```text
love
```

的时候不能提前看到后面的：

```text
AI
```

所以使用 Causal Mask：

```text
Token1  √ × × ×
Token2  √ √ × ×
Token3  √ √ √ ×
Token4  √ √ √ √
```

当前 Token 只能关注：

> **自己以及前面的 Token。**

这使模型能够进行自回归生成。

---

# 十五、LLM 是怎么生成一句话的？

这是非常值得理解的。

用户：

```text
中国的首都是
```

模型不是直接一次性生成完整答案。

而是：

```text
中国的首都是
      ↓
预测下一个 Token
      ↓
北京
```

然后：

```text
中国的首都是北京
        ↓
预测下一个 Token
        ↓
。
```

本质：

\[
P(x_t|x_1,x_2,...,x_{t-1})
\]

即：

> 根据前面所有 Token，预测下一个 Token 的概率分布。

所以 LLM 本质上仍然是：

```text
Next Token Prediction
```

只是规模、训练数据和训练方式使这种能力最终涌现出了复杂的语言理解、推理、代码等能力。

---

# 十六、为什么 Transformer 比 RNN 更适合 LLM？

这是经典面试题。

RNN：

```text
Token1
 ↓
Token2
 ↓
Token3
 ↓
Token4
```

天然偏顺序处理。

Transformer：

```text
Token1 ─┬─────────┐
Token2 ─┼─ Attention
Token3 ─┼─────────┤
Token4 ─┴─────────┘
```

主要优势：

**① 训练阶段并行化能力强**

相比 RNN 的逐步递归，更适合 GPU 大规模训练。

**② 更容易建模长距离依赖**

两个相距很远的 Token 可以直接通过 Attention 建立联系。

**③ 可扩展性强**

非常适合扩大：

```text
Parameters
Data
Compute
```

因此成为现代 LLM 的核心架构。

---

# 十七、Attention 最大的问题是什么？

非常重要。

标准 Self-Attention 对长度为 `n` 的序列，需要构造近似：

```text
n × n
```

的 Attention 关系。

因此经典 Attention 的计算/内存开销随序列长度增长很快，通常讨论为：

\[
O(n^2)
\]

所以：

```text
Context 越长
↓
Attention 计算越来越贵
↓
显存 / 延迟 / 成本上升
```

这和 Agent 开发非常相关。

因为 Agent 特别容易产生：

```text
System Prompt
+
Conversation
+
RAG Documents
+
Tool Results
+
Memory
+
Skills
+
Agent History
```

最终 Context 越来越大。

所以你之前学的：

> **Context Engineering / Harness Engineering**

和 Transformer 的底层限制其实直接相关。

---

# 十八、KV Cache 是什么？

Agent 岗有一定概率问，尤其偏推理工程时。

假设已经生成：

```text
I love artificial
```

现在生成：

```text
intelligence
```

前面 Token 的 Key / Value 很多计算其实已经做过了。

如果每生成一个 Token 都重新算：

```text
I
I love
I love artificial
...
```

非常浪费。

所以推理时可以缓存之前 Token 的：

```text
K
V
```

这就是：

> **KV Cache**

作用：

> **避免自回归生成过程中重复计算历史 Token 的 Key / Value，提高推理效率。**

代价是：

```text
Context 越长
↓
KV Cache 越大
↓
显存占用增加
```

---

# 十九、Transformer 和 Agent 到底有什么关系？

这是你面试真正应该讲清楚的。

```text
Transformer
     ↓
提供底层神经网络架构
     ↓
LLM
     ↓
理解 + 推理 + 生成
     ↓
Agent
     ↓
┌───────────────────────┐
│ LLM                   │
│ Tools                 │
│ Skills                │
│ Memory                │
│ Context               │
│ Runtime / Harness     │
└───────────────────────┘
```

所以：

> **Transformer 解决模型内部怎么处理 Token 和上下文；Agent 解决怎么把模型和外部工具、知识、状态以及执行环境组织起来完成真实任务。**

不要说：

> Agent 是 Transformer。

而应该：

> Agent 的“大脑”通常是基于 Transformer 的 LLM，但 Agent 本身是更上层的系统工程概念。

---

# 二十、Agent 开发面试最可能问你的 Transformer 问题

如果你面的是 **AI Agent / AI 应用开发岗，而不是大模型算法岗**，我建议按下面优先级准备。

### 第一梯队：必须会

| 问题 | 你至少要回答到 |
|---|---|
| Transformer 是什么？ | Attention 为核心的序列建模架构 |
| Self-Attention 是什么？ | Token 根据相关性聚合其他 Token 信息 |
| Q/K/V 分别是什么？ | Query 查询、Key 匹配、Value 信息 |
| Multi-Head Attention 为什么需要？ | 不同子空间学习不同关系 |
| Transformer 为什么比 RNN 更适合 LLM？ | 并行训练、长距离依赖、扩展性 |
| GPT 为什么能生成文本？ | Decoder-only + causal attention + next-token prediction |
| Token 是什么？ | 模型处理文本的基本单位 |
| Embedding 是什么？ | 将离散 Token/文本映射为向量表示 |
| Context Window 是什么？ | 一次推理能处理的上下文 Token 范围 |
| Transformer 和 LLM、Agent 什么关系？ | 底层架构 → 模型 → 应用系统 |

### 第二梯队：Agent 岗非常值得会

| 问题 | 核心 |
|---|---|
| 为什么 Agent 要做 Context Engineering？ | Context 有限制且越长成本越高 |
| 为什么不能把所有 Tool 都塞给模型？ | Context 污染、选择难度、Token 成本 |
| 为什么 Skill 要渐进式加载？ | 减少无关 Context |
| RAG 为什么需要 Embedding？ | 语义检索 |
| Token 为什么影响 Agent 成本？ | 输入/输出及长上下文都影响推理资源与计费 |
| KV Cache 是什么？ | 缓存历史 K/V，提高自回归推理效率 |
| Attention 为什么长上下文贵？ | 标准 Attention 的二次复杂度问题 |
| Tool Calling 和 Transformer 什么关系？ | 本质上模型仍在生成符合约束的结构化输出 |

---

# 二十一、几个很可能出现的追问题

### Q1：Function Calling 是不是 LLM 真的执行函数？

不是。

模型基于：

```text
User Prompt
+
Tool Description
+
Tool Schema
+
Context
```

生成类似：

```json id="62tj04"
{
  "name": "query_database",
  "arguments": {
    "sql": "..."
  }
}
```

本质上仍然是模型生成结构化输出。

真正执行的是：

```text
Runtime / Application
```

这正好能把你之前学的 Function Calling 和 Transformer 串起来。

---

### Q2：为什么 Tool Description 写不好会选错工具？

因为 LLM 选择 Tool 的时候，Tool 的：

```text
Name
Description
Schema
```

本身也成为模型上下文的一部分。

如果：

```text
Tool A: search_data
Tool B: query_data
Tool C: find_data
```

描述高度重叠，模型就更难判断。

所以 Tool Design 本质上也是：

> **Context Engineering。**

---

### Q3：为什么 Agent Context 不能无限增长？

因为：

```text
Agent Loop
 ↓
Conversation
 ↓
Tool Result
 ↓
RAG
 ↓
Memory
 ↓
越来越长
```

最终受到：

```text
Context Window
Attention / KV Cache 开销
Latency
Token Cost
有效信息密度
```

等问题影响。

所以需要：

```text
Summarization
Compaction
Retrieval
Tool Result Truncation
Progressive Disclosure
External State
```

---

### Q4：RAG 和 Transformer 有什么关系？

可以回答：

> LLM 的 Transformer 负责处理当前 Context，但模型参数中的知识存在时效性和覆盖范围限制。RAG 在模型推理前先通过检索系统找到外部知识，再把相关内容加入 Context，让 Transformer 基于这些内容完成生成。

流程：

```text
Question
 ↓
Embedding
 ↓
Vector Search
 ↓
Relevant Documents
 ↓
Prompt / Context
 ↓
Transformer LLM
 ↓
Answer
```

---

# 二十二、你准备 Agent 岗，Transformer 学到什么程度够？

我建议你做到这三个层级。

**第一层必须能画：**

```text
Text
 ↓
Tokenizer
 ↓
Embedding + Position
 ↓
Transformer Block
 ├─ Self-Attention
 ├─ Add & Norm
 ├─ FFN
 └─ Add & Norm
 ↓
重复 N 层
 ↓
预测 Next Token
```

**第二层必须能解释：**

```text
Attention
Q / K / V
Self-Attention
Multi-Head
Causal Mask
Embedding
Context Window
KV Cache
```

**第三层必须能和 Agent 联系：**

```text
Transformer
 ↓
Context 有成本/限制
 ↓
Context Engineering

LLM
 ↓
不能直接操作外部世界
 ↓
Tool Use

Tool Schema
 ↓
进入模型 Context
 ↓
Function Calling

大量 Skills / Tools
 ↓
Context 膨胀
 ↓
Retrieval + Progressive Disclosure

长期 Agent
 ↓
历史越来越多
 ↓
Compaction + Memory + Harness
```

对于 **Agent 应用开发岗**，最后这一层往往比让你手推 Attention 公式更有价值。

---

# 最终速记

```text
Transformer
│
├── Tokenization
│     └─ 文本 → Token
│
├── Embedding
│     └─ Token → Vector
│
├── Position
│     └─ 告诉模型 Token 顺序
│
├── Self-Attention
│     ├─ Q：我想找什么
│     ├─ K：我有什么特征用于匹配
│     └─ V：我真正提供什么信息
│
├── Multi-Head Attention
│     └─ 从多个表示子空间建模关系
│
├── FFN
│     └─ 进一步加工 Token 表示
│
├── Residual + Norm
│     └─ 帮助深层网络稳定训练
│
└── Decoder
      ├─ Causal Attention
      └─ Next Token Prediction
```

最后把你最近学的东西全部串成一条主线：

```text
Transformer
     ↓
     LLM
     ↓
推理 / 生成 / Tool Selection
     ↓
Function Calling
     ↓
Runtime / Harness
     ↓
Tool Use
     ↓
MCP
     ↓
外部系统
```

你面 Agent 开发岗时，**Transformer 要证明你理解 LLM 为什么能工作；Function Calling/MCP/RAG/Context/Harness 要证明你知道怎么把这个 LLM 做成真正能落地的 Agent 系统。**