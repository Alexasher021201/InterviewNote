可以把 **RAG（Retrieval-Augmented Generation，检索增强生成）** 理解成 Agent / LLM 开发里非常核心的一套技术。面试时建议你不仅知道定义，还要能讲清楚**完整链路、为什么需要它、每一步有什么坑**。

## 1. RAG 是什么

RAG = **Retrieval（检索） + Generation（生成）**。

核心思想：

> **先从外部知识库中找到和用户问题相关的资料，再把资料和问题一起交给 LLM，让 LLM 基于这些资料回答。**

例如公司有大量内部文档：

```text
用户：
“公司的年假有多少天？”
        ↓
   搜索公司知识库
        ↓
找到《员工休假管理制度》
        ↓
提取相关段落
        ↓
问题 + 相关段落 → LLM
        ↓
LLM 根据资料回答
```

所以 RAG 本质上是在解决：

**LLM 自己不知道 / 记不准 → 先给它找资料，再让它回答。**

---

# 2. 为什么需要 RAG？

大模型本身存在几个问题：

| 问题 | RAG 如何解决 |
|---|---|
| 不知道公司内部知识 | 查询企业私有知识库 |
| 训练数据有截止时间 | 接入最新文档 |
| 容易产生幻觉 | 提供真实资料作为依据 |
| 无法知道知识来源 | 可以返回引用来源 |
| Fine-tuning 成本较高 | 修改知识库即可更新知识 |

这里面有一个非常高频的面试点：

### RAG ≠ Fine-tuning

简单记：

```text
RAG
→ 给模型“查资料”

Fine-tuning
→ 给模型“训练/学习”
```

例如：

> “让模型知道公司最新报销政策。”

更适合 **RAG**。

> “让模型长期按照公司的客服风格回答。”

可能更适合 **Fine-tuning**。

---

# 3. RAG 的完整流程 ⭐

这是面试最应该掌握的部分。

完整 RAG 通常分成两个阶段：

```text
        【离线阶段】

文档 PDF / Word / 网页
          ↓
      Document Loader
          ↓
      文档解析
          ↓
       Chunking
          ↓
      Embedding
          ↓
      Vector DB


        【在线阶段】

用户 Question
          ↓
      Embedding
          ↓
   Vector Search
          ↓
      Top-K Chunks
          ↓
      （Rerank）
          ↓
Question + Context
          ↓
         LLM
          ↓
       Answer
```

把这张流程图记住，基本就掌握了 RAG 的主干。

---

# 4. RAG 常用核心概念

### ① Document Loader

负责读取原始数据，例如：

```text
PDF
Word
网页
Markdown
数据库
Notion
```

最终统一转换成可以处理的文本 / Document。

---

### ② Chunking ⭐

不能直接把一本 300 页 PDF 全塞给 LLM。

所以需要：

```text
完整文档
   ↓
切成很多小段
   ↓
Chunk 1
Chunk 2
Chunk 3
...
```

每一段就叫 **Chunk**。

这里通常有两个重要参数：

```text
Chunk Size
```

每块多大。

以及：

```text
Chunk Overlap
```

相邻 Chunk 重复多少内容。

例如：

```text
Chunk 1：
AAAA BBBB CCCC

Chunk 2：
          CCCC DDDD EEEE
          ↑
        overlap
```

Overlap 可以减少**一句话刚好被切断导致语义丢失**的问题。

---

### ③ Embedding ⭐⭐⭐

Embedding 是 RAG 最核心的概念之一。

Embedding Model 会把文本转换成一个**向量**：

```text
"苹果手机"
      ↓
Embedding Model
      ↓
[0.21, -0.72, 0.34, 0.91, ...]
```

这个向量代表文本的**语义信息**。

因此：

```text
"怎么申请退款？"

和

"商品如何退货退款？"
```

虽然文字不完全一样，但 Embedding 可能比较接近。

---

# 5. Vector Database（向量数据库）

Embedding 生成的向量通常存进 **Vector DB**。

常见的有：

```text
FAISS
Milvus
Pinecone
Weaviate
Qdrant
Chroma
```

数据库里大概是：

```text
Chunk                     Vector

公司年假规定...       [0.13, 0.72, ...]
公司报销流程...       [0.91, 0.24, ...]
员工保险政策...       [0.37, 0.68, ...]
```

用户提问：

```text
“员工一年可以休几天？”
```

也转换成：

```text
[0.15, 0.70, ...]
```

然后寻找**距离最近 / 相似度最高的向量**。

这就是：

**Vector Search / Semantic Search（语义检索）**。

---

# 6. Similarity Search

怎么判断两个向量像不像？

常见方法：

- **Cosine Similarity**
- Dot Product
- Euclidean Distance

面试最常见的是：

### Cosine Similarity（余弦相似度）

核心思想是比较两个向量的**方向是否接近**。

越接近：

```text
用户 Query
     ↓
Embedding
     ↓
寻找最相似的 Chunk
```

然后返回：

```text
Top-K
```

例如：

```text
Top-3

Chunk 17   similarity = 0.91
Chunk 42   similarity = 0.87
Chunk 8    similarity = 0.82
```

---

# 7. Top-K

Top-K 就是：

> **检索最相关的 K 个 Chunk。**

例如：

```text
Top-K = 5
```

意味着把最相关的 5 个 Chunk 给 LLM。

但注意：

**K 并不是越大越好。**

太小：

```text
可能漏掉关键信息
```

太大：

```text
Context 噪声增加
Token 成本增加
LLM 更难找到重点
```

这也是一个不错的面试点。

---

# 8. Reranking ⭐⭐

实际项目通常不会：

```text
Vector Search → 直接给 LLM
```

而可能：

```text
Query
 ↓
Vector Search
 ↓
召回 Top 20
 ↓
Reranker
 ↓
重新排序
 ↓
选择 Top 5
 ↓
LLM
```

为什么？

因为 Embedding 检索的目标更偏向：

> **“尽量别漏掉可能相关的内容。”**

Reranker 再负责：

> **“这些内容里面，到底哪个最相关？”**

所以可以理解成：

```text
Retriever → 粗筛

Reranker → 精排
```

---

# 9. Hybrid Search ⭐⭐

纯 Vector Search 有时候不够。

例如用户查询：

```text
ERROR_CODE_0x8007
```

这种精确关键词，Embedding 不一定特别擅长。

所以实际系统经常使用：

### Hybrid Search

```text
Vector Search
      +
Keyword Search / BM25
      ↓
合并结果
      ↓
Rerank
```

简单理解：

```text
Vector Search
擅长“意思相似”

BM25
擅长“关键词匹配”
```

两者结合通常更加稳定。

---

# 10. 一个比较完整的生产级 RAG

你可以把前面的知识组合起来：

```text
              Documents
                  ↓
              Chunking
                  ↓
              Embedding
                  ↓
              Vector DB
                  │
                  │
User Query        │
    ↓             │
Query Rewrite     │
    ↓             │
Hybrid Search ←───┘
    ↓
Top 20 Documents
    ↓
Reranker
    ↓
Top 5 Documents
    ↓
Prompt Construction
    ↓
Question + Context
    ↓
       LLM
        ↓
Answer + Citation
```

这个流程在面试中非常值得记住。

---

# 11. 面试重点：RAG 最难的是什么？ ⭐⭐⭐

很多初学者会觉得：

> RAG 最重要的是 Vector DB。

其实真正项目里，**检索质量通常才是核心问题之一**。

如果 Retrieval 找错了：

```text
错误资料
   ↓
交给 LLM
   ↓
LLM 再强
   ↓
也可能得到错误答案
```

也就是：

> **Garbage In → Garbage Out**

因此优化 RAG 时，经常优化：

```text
Chunking
    ↓
Embedding Model
    ↓
Query Rewrite
    ↓
Hybrid Search
    ↓
Top-K
    ↓
Reranking
    ↓
Prompt
```

而不只是换一个更强的 LLM。

---

# 12. RAG 面试高频问题

下面这些建议你重点准备：

| 面试问题 | 你至少要知道什么 |
|---|---|
| 什么是 RAG？ | Retrieval + Generation |
| 为什么需要 RAG？ | 私有知识、实时知识、降低幻觉 |
| RAG 完整流程？ | Chunk → Embedding → Retrieve → Rerank → LLM |
| 什么是 Embedding？ | 文本 → 语义向量 |
| 为什么需要 Chunk？ | 长文档无法直接检索/塞入 Context |
| Chunk Size 怎么选？ | 太小语义不完整，太大检索不精准 |
| Chunk Overlap 为什么存在？ | 防止切断上下文 |
| Vector Search 是什么？ | 根据向量相似度检索 |
| Top-K 怎么选择？ | Recall、噪声、Token 成本之间权衡 |
| Reranker 是什么？ | 对召回结果进一步精排 |
| Hybrid Search 是什么？ | Vector + BM25/关键词 |
| RAG 和 Fine-tuning 区别？ | 查资料 vs 调整模型行为/能力 |
| RAG 如何降低幻觉？ | Grounding + Citation，但不能完全消除 |
| RAG 效果不好怎么办？ | 分析检索、切块、Embedding、Rerank、Prompt |

---

## 最后建立一个整体认知

你现在准备 Agent 开发，可以把这些概念串起来：

```text
                    AI Agent
                       │
            ┌──────────┴──────────┐
            ↓                     ↓
           LLM                   Tool
                                  │
                    ┌─────────────┴────────────┐
                    ↓                          ↓
                   RAG                        API
                    │
            Retriever / Vector DB
                    │
              企业知识库


LangChain / LangGraph
        ↓
负责把上面的组件组织起来
```

所以 **RAG 本身不是 Agent**，它更像是 Agent 可以使用的一种**知识获取能力**。

面试阶段尤其建议你牢牢记住这条主线：

> **Document → Chunk → Embedding → Vector DB → Query → Retrieval → Rerank → Context + Prompt → LLM → Answer**

能把这条链路每一步的**作用、为什么需要、可能有什么问题**讲清楚，RAG 基础面试基本就比较扎实了。