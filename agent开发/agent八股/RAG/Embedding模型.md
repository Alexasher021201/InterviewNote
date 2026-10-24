**面试准备可以先记住这几个系列：BGE、Qwen Embedding、OpenAI Embedding、E5、Jina、Cohere 和 Voyage。**它们都是把文本等内容转换成向量的模型，但部署方式、语言能力和成本不同。

**1．可以本地部署的代表模型**

| 模型系列 | 代表型号 | 特点与适用场景 |
|---|---|---|
| **BGE 中文系列** | `bge-small-zh-v1.5`、`bge-base-zh-v1.5`、`bge-large-zh-v1.5` | 面向中文文本；提供不同规模，可作为中文知识库的评估基线。[官方模型卡](https://huggingface.co/BAAI/bge-large-zh-v1.5) |
| **BGE-M3** | `bge-m3` | 支持多语言、最长 8192 tokens；除稠密向量外，还支持稀疏和多向量检索。[官方模型卡](https://huggingface.co/BAAI/bge-m3) |
| **Qwen3 Embedding** | `Qwen3-Embedding-0.6B`、`4B`、`8B` | 支持多语言与任务指令，可在模型规模和推理开销之间选择。[官方模型卡](https://huggingface.co/Qwen/Qwen3-Embedding-0.6B) |
| **E5** | `multilingual-e5-large` | 多语言文本模型；使用时需要按要求添加 `query:` 和 `passage:` 前缀。[官方模型卡](https://huggingface.co/intfloat/multilingual-e5-large) |
| **Jina Embeddings** | `jina-embeddings-v3`、`jina-embeddings-v4`，以及 v5 text 系列 | v3 面向多语言文本，v4 支持文本与图像；也有 API 服务。不同型号需分别查看许可证。[v3](https://huggingface.co/jinaai/jina-embeddings-v3)、[v4](https://huggingface.co/jinaai/jina-embeddings-v4)、[v5 text](https://huggingface.co/jinaai/jina-embeddings-v5-text-small) |

这里的“可以本地部署”指可以获取模型权重并自行运行，商用条件仍以各型号的许可证为准。

**2．通过商业 API 调用的代表模型**

| 厂商 | 代表型号 | 特点 |
|---|---|---|
| **OpenAI** | `text-embedding-3-small`、`text-embedding-3-large` | small 更侧重成本，large 更侧重能力；默认分别输出 1536、3072 维，也支持缩短维度。[OpenAI 官方文档](https://developers.openai.com/api/docs/guides/embeddings) |
| **阿里云通义** | `text-embedding-v4` | 基于 Qwen3 的多语言文本向量模型，支持自定义输出维度。[官方文档](https://www.alibabacloud.com/help/en/model-studio/text-embedding-v4) |
| **Cohere** | `embed-v4.0` | 支持文本、图像及图文混合输入，可用于包含视觉内容的文档检索。[官方文档](https://docs.cohere.com/v2/docs/cohere-embed) |
| **Voyage AI** | `voyage-4`、`voyage-4-large`、`voyage-4-lite`、`voyage-code-4` | 提供通用、多语言及代码检索型号，也有侧重延迟和成本的版本。[官方文档](https://docs.voyageai.com/docs/embeddings) |

**3．对于你准备 Agent 面试，建议重点理解三个系列**

- **BGE**：掌握中文 Embedding，以及 BGE-M3 的多种检索表示。
- **Qwen3 Embedding**：理解本地部署时，模型规模、显存、吞吐量和效果的取舍。
- **OpenAI Embedding**：理解 API 接入、输出维度、调用成本和索引构建。

其他系列知道定位即可。**比背很多型号更重要的是能解释“为什么选它”。**

例如，面试官问“中文企业知识库用哪个”，你可以回答：

> “我会把 BGE 中文模型、BGE-M3 和 Qwen3 Embedding 放入候选集，用真实中文业务问题比较 Recall@K、排序质量、延迟和成本。如果允许使用外部 API，也会加入商业模型作为对照。最终按评估结果选择。”

**4．三个容易混淆的概念一定要分清**

| 组件 | 输入 | 输出 | 作用 |
|---|---|---|---|
| **Embedding 模型** | 一段文本 | 一个向量，或多种向量表示 | 为检索建立表示 |
| **Reranker 模型** | 问题＋候选文档 | 相关性分数或排序 | 精排已检索到的候选 |
| **生成模型／LLM** | 问题＋参考资料 | 自然语言答案 | 组织并生成回答 |

典型链路是：

```text
Embedding 编码 → 向量检索 → Reranker 精排 → LLM 生成答案
```

**最后记住一个很重要的工程细节：查询向量和文档向量必须处于兼容的向量空间。**

通常要使用同一套 Embedding 模型及配套配置，按官方要求分别编码查询和文档。**即使两个模型都输出 1024 维，也不能直接混用。**更换为不兼容的模型时，通常需要重新编码文档并重建向量索引。