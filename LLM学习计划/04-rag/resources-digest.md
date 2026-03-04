# 阶段 4 推荐资源深度总结

> 本文将 [阶段 4：RAG 检索增强生成](./index.md) 中推荐的核心资源提炼为一篇通俗易懂的中文解读。
> 目标：即使你不阅读原始英文资源，也能建立对 RAG 技术的完整理解。

---

## 目录

- [4.1 RAG from Scratch：从零实现 RAG](#41-rag-from-scratch从零实现-rag)
- [4.2 IBM RAG 科普视频：什么是检索增强生成](#42-ibm-rag-科普视频什么是检索增强生成)
- [4.3 Pinecone 分块策略详解](#43-pinecone-分块策略详解)
- [4.4 ChromaDB：最简单的向量数据库](#44-chromadb最简单的向量数据库)
- [4.5 Qdrant：生产级向量数据库](#45-qdrant生产级向量数据库)
- [4.6 LlamaIndex 高级 RAG 技巧](#46-llamaindex-高级-rag-技巧)
- [总结：串联所有知识点](#总结串联所有知识点)

---

## 4.1 RAG from Scratch：从零实现 RAG

> 原文：[RAG from Scratch](https://github.com/langchain-ai/rag-from-scratch)（LangChain 团队）

这是 LangChain 官方推出的「从零实现 RAG」系列教程，配套有 Jupyter 笔记本和视频，非常适合想动手搭建第一个 RAG 系统的初学者。

### 为什么要学 RAG？

大语言模型（LLM）的训练数据是固定的，有两个明显局限：

1. **知识截止**：模型不知道训练之后发生的事，也无法访问你的私有数据（公司文档、产品手册等）
2. **微调不划算**：用微调来补充事实性知识效果往往不好，而且成本高

RAG（检索增强生成）提供了一种更实用的方案：在生成回答之前，先从外部知识库检索相关文档，再把这些文档和用户问题一起喂给 LLM。这样既利用了 LLM 的语言能力，又保证了答案基于真实、可追溯的数据。

### 教程涵盖的核心环节

教程从零开始，把 RAG 拆成三个主要步骤：

| 环节 | 作用 | 通俗理解 |
|------|------|----------|
| **索引（Indexing）** | 把文档处理成可检索的形式 | 把书整理进图书馆，建好目录 |
| **检索（Retrieval）** | 根据用户问题找到相关文档 | 根据问题去图书馆找对应的书 |
| **生成（Generation）** | 基于检索到的内容生成回答 | 拿着找到的书，组织成答案写出来 |

### 数据流示意

```
文档 → 分块 → 向量化（Embedding）→ 存入向量数据库
                                    ↓
用户问题 → 向量化 → 相似度搜索 → 检索到相关文档块
                                    ↓
        检索到的文档 + 用户问题 → 送入 LLM → 生成最终回答
```

### 如何学习这个资源？

- **仓库**：GitHub 上有配套的 Jupyter 笔记本，可以边看边跑
- **视频**：YouTube 有完整视频系列，讲解每个步骤的实现
- **建议**：先跟着视频或笔记本跑通一遍基础流程，再尝试修改参数（如分块大小、Top-K）观察效果变化

---

## 4.2 IBM RAG 科普视频：什么是检索增强生成

> 原文：[What is Retrieval-Augmented Generation (RAG)?](https://www.youtube.com/watch?v=T-D1OfcDW1M)（IBM Technology，YouTube）

这是 IBM 出品的 RAG 科普视频，用通俗语言解释 RAG 是什么、为什么重要、能解决什么问题。适合没有技术背景的人快速建立概念。

### RAG 是什么？

RAG 是一种 AI 框架，让大语言模型在回答问题时，能够从外部知识库中检索事实，并用这些事实来支撑自己的回答。可以把它理解成「开卷考试」：传统 LLM 是闭卷，只能靠训练时记住的内容；RAG 让模型在答题前先去「翻书」（检索），再基于找到的内容作答。

### 为什么需要 RAG？

| 问题 | RAG 的解决方式 |
|------|----------------|
| 模型知识过时 | 从外部数据源获取最新信息 |
| 模型会「一本正经地胡说」 | 用可验证的外部事实来约束回答 |
| 模型不知道你的私有数据 | 把公司文档、产品手册等接入知识库 |
| 持续微调成本高 | 只需更新知识库，无需重新训练模型 |

### RAG 带来的好处

- **减少幻觉**：用可验证的外部事实约束模型，减少编造
- **降低成本**：不需要频繁微调，只需维护和更新知识库
- **提高可信度**：用户可以看到答案来自哪些文档，便于核实
- **保护数据**：敏感信息不必写入模型参数，可放在受控的知识库中

### 适合谁看？

如果你对 RAG 完全没概念，想先搞懂「它是什么、能干什么」，这个视频是很好的入门。看完后再去学具体实现（如 RAG from Scratch）会更容易理解每一步在解决什么问题。

---

## 4.3 Pinecone 分块策略详解

> 原文：[Chunking Strategies for LLM Applications](https://www.pinecone.io/learn/chunking-strategies/)（Pinecone 官方博客）

分块（Chunking）是 RAG 中最容易被忽视、但对效果影响极大的环节。这篇文章系统讲解了为什么要分块、有哪些分块方法、如何选择适合自己场景的策略。

### 什么是分块？为什么必须分块？

**分块**就是把长文档切成小段（chunk）。原因主要有两个：

1. **嵌入模型的限制**：每个嵌入模型都有上下文窗口（context window），超过限制的文本会被截断，重要信息可能丢失
2. **检索需要**：检索是按「块」来比较相似度的。如果块太大，检索不精准；如果块太小，上下文不足，检索到的内容可能没有意义

**经验法则**：如果一段文字在人看来「脱离上下文也能懂」，那对语言模型来说通常也能懂。所以分块的目标是：**每块既包含足够的信息，又能在没有 surrounding context 的情况下被理解**。

### 分块在语义搜索中的作用

在语义搜索中，文档会被分块后存入向量数据库。用户提问时，用问题的向量去和每个块的向量比较相似度，返回最相似的块。分块策略直接影响：

- 块太大 → 检索到的块可能包含大量无关内容，噪音多
- 块太小 → 可能把一个完整概念切断，检索到的块缺乏上下文

### 长上下文模型还需要分块吗？

即使用 200K Token 的长上下文模型，不分块也会有问题：

- **成本与延迟**：大块会增加下游 LLM 的 Token 消耗和响应时间
- **Lost in the Middle**：研究表明，模型容易忽略长文档中间的内容，即使这些内容在上下文中。通过分块和精选，只把最相关的信息传给 LLM，能提高质量

### 选择分块策略时要考虑什么？

| 因素 | 说明 |
|------|------|
| **检索结果怎么用** | 语义搜索、问答、RAG、Agent 等场景对块的大小和结构要求不同 |
| **用户问题的特点** | 问题短而具体，还是长而复杂？这会影响块和查询的匹配方式 |
| **嵌入模型** | 不同模型对文本长度、领域（代码、金融、医疗等）的适配不同 |
| **数据类型** | 长文章、书籍 vs 短消息、商品描述，分块策略会不同 |

### 常见分块方法

#### 1. 固定大小分块（Fixed-size Chunking）

最直接的方式：按 Token 数或字符数切分。块大小通常参考嵌入模型的最大上下文（如 1024、8196 Token）。

- **优点**：实现简单，大多数场景下够用
- **缺点**：可能切断语义边界（如把一个句子从中间劈开）
- **建议**：先从这里开始，只有在效果不够好时再尝试更复杂的策略

#### 2. 内容感知分块（Content-aware Chunking）

利用文档结构来分块，而不是机械地按长度切：

- **按句子/段落切**：用 spaCy、NLTK 等工具识别句子边界
- **递归字符分块**：LangChain 的 `RecursiveCharacterTextSplitter` 会按 `\n\n`、`\n`、空格等分隔符依次尝试，在保证块大小的前提下尽量在自然边界处切分

#### 3. 按文档结构分块（Document structure-based）

针对不同格式使用专门的分块逻辑：

- **Markdown**：按标题、列表、代码块等结构切分
- **LaTeX**：按章节、公式等逻辑单元切分
- **HTML**：利用 `<p>`、`<h1>` 等标签
- **PDF**：需要处理页眉、表格、图片等，通常需要专门的解析工具

#### 4. 语义分块（Semantic Chunking）

利用嵌入模型来识别「主题切换」的位置：把文档切成句子，将相邻句子分组并计算嵌入，通过比较相邻组的语义距离，在主题变化处切分。这样每个块内部语义更一致。

#### 5. 上下文分块（Contextual Chunking）

当文档非常长、主题切换频繁时，单块可能丢失全局上下文。Anthropic 提出的做法是：用 LLM 为每个块生成一个「上下文描述」（概括整篇文档对该块的背景），把这个描述附加到块上再嵌入。这样检索时能兼顾局部和全局信息。

### 分块后的扩展：Chunk Expansion

分块策略不是一成不变的。检索时，可以对每个检索到的块做**扩展**：把相邻的块（如前后的段落、整页、甚至整篇文档）一起取出来，给 LLM 更多上下文。这样既能保持检索时的小块（提高精准度），又能在生成时提供足够的上下文。

### 如何找到最佳分块大小？

1. **多索引/多命名空间**：用不同块大小建多个索引，用代表性查询做对比测试
2. **迭代调参**：从 128、256、512、1024 等 Token 数中选一个范围，结合你的内容类型和嵌入模型反复试验
3. **评估指标**：用检索质量、生成答案的相关性等指标来评判哪种块大小更好

---

## 4.4 ChromaDB：最简单的向量数据库

> 原文：[ChromaDB 官方文档](https://docs.trychroma.com/)

ChromaDB 是一个开源的向量数据库，专为存储和检索向量嵌入设计。它最大的特点是**简单**：安装快、上手快，非常适合本地开发和原型验证。

### ChromaDB 是什么？

向量数据库和传统数据库的区别在于：传统数据库按关键词搜索（搜「苹果」只能找到包含「苹果」的文档），向量数据库按**语义**搜索（搜「水果」也能找到关于「苹果」的文档）。ChromaDB 就是专门做这件事的——存向量、做相似度搜索。

### 核心特性

| 特性 | 说明 |
|------|------|
| **安装简单** | `pip install chromadb` 即可 |
| **检索能力** | 向量搜索、全文搜索、元数据过滤、多模态检索 |
| **可扩展** | 本地可用 DuckDB，大规模可用 ClickHouse 等后端 |
| **多语言** | Python、JavaScript/TypeScript、Ruby、PHP、Java |
| **生态集成** | 支持 HuggingFace、OpenAI、Google 的嵌入模型，兼容 LangChain、LlamaIndex |

### 典型工作流

1. **创建集合（Collection）**：相当于一张表，用来存某一类文档的向量
2. **添加文档**：把文本、嵌入向量和元数据（如文件名、页码）存入集合
3. **相似度搜索**：用查询文本的向量去集合里找最相似的文档块
4. **提供给 LLM**：把检索到的文档作为上下文，和用户问题一起传给 LLM 生成回答

### 适合谁用？

- **学习 RAG**：零配置，几分钟就能跑通一个语义搜索 demo
- **原型开发**：不需要部署独立服务，适合快速验证想法
- **小规模应用**：文档量不大、对高可用要求不高的场景

如果后续要上生产、数据量大、需要分布式部署，可以考虑 Qdrant、Pinecone 等更面向生产的方案。

---

## 4.5 Qdrant：生产级向量数据库

> 原文：[Qdrant 官方文档](https://qdrant.tech/documentation/)

Qdrant 是一个面向 AI 的向量数据库和语义搜索引擎，开源、可自托管，也提供云服务。相比 ChromaDB，Qdrant 更侧重**生产环境**：高性能、可扩展、支持过滤和分布式部署。

### Qdrant 是什么？

Qdrant 帮助从非结构化数据中做语义检索。你可以把文档向量化后存入 Qdrant，然后用自然语言或向量进行查询，找到语义最相关的内容。

### 快速上手：本地用 Docker 运行

```bash
# 拉取镜像
docker pull qdrant/qdrant

# 运行服务（数据会存在 ./qdrant_storage）
docker run -p 6333:6333 -p 6334:6334 \
    -v "$(pwd)/qdrant_storage:/qdrant/storage:z" \
    qdrant/qdrant
```

- REST API：`http://localhost:6333`
- Web 控制台：`http://localhost:6333/dashboard`
- gRPC：`localhost:6334`

### 核心概念与操作

#### 1. 创建集合（Collection）

集合用来存储向量数据。创建时需要指定向量维度和相似度度量（如点积 Dot、余弦 Cosine）：

```python
from qdrant_client import QdrantClient
from qdrant_client.models import Distance, VectorParams

client = QdrantClient(url="http://localhost:6333")
client.create_collection(
    collection_name="test_collection",
    vectors_config=VectorParams(size=4, distance=Distance.DOT),
)
```

#### 2. 插入向量和元数据（Payload）

每个向量可以附带 payload（如城市名、文件名、标签等），用于后续过滤：

```python
from qdrant_client.models import PointStruct

client.upsert(
    collection_name="test_collection",
    points=[
        PointStruct(id=1, vector=[0.05, 0.61, 0.76, 0.74], payload={"city": "Berlin"}),
        PointStruct(id=2, vector=[0.19, 0.81, 0.75, 0.11], payload={"city": "London"}),
        # ...
    ],
)
```

#### 3. 向量搜索

用查询向量搜索最相似的 Top-K 结果：

```python
search_result = client.query_points(
    collection_name="test_collection",
    query=[0.2, 0.1, 0.9, 0.7],
    limit=3
).points
```

#### 4. 带过滤的搜索

Qdrant 支持在向量搜索的基础上加过滤条件，例如只查 `city="London"` 的结果：

```python
from qdrant_client.models import Filter, FieldCondition, MatchValue

search_result = client.query_points(
    collection_name="test_collection",
    query=[0.2, 0.1, 0.9, 0.7],
    query_filter=Filter(
        must=[FieldCondition(key="city", match=MatchValue(value="London"))]
    ),
    with_payload=True,
    limit=3,
).points
```

### 生产级能力

| 能力 | 说明 |
|------|------|
| **分布式部署** | 支持多节点、高可用、故障恢复 |
| **多租户** | 支持数据隔离、分区，适合多用户场景 |
| **向量量化** | 通过量化技术提升搜索速度和内存效率 |
| **云服务** | Qdrant Cloud 提供托管版，免运维 |

### 适合谁用？

- 需要**过滤**（按元数据筛选）的 RAG 场景
- 数据量大、需要**分布式**和**高可用**的生产环境
- 希望自托管或使用云托管的企业用户

---

## 4.6 LlamaIndex 高级 RAG 技巧

> 原文：[Advanced RAG Techniques](https://www.llamaindex.ai/blog/a-cheat-sheet-and-some-recipes-for-building-advanced-rag-803a9d94c41b)（LlamaIndex 官方博客，亦有 [Medium 版本](https://medium.com/llamaindex-blog/a-cheat-sheet-and-some-recipes-for-building-advanced-rag-803a9d94c41b)）

这篇文章提供了一份「高级 RAG 技巧清单」，并配有 LlamaIndex 的代码示例。适合已经搭好基础 RAG、想从 60 分提升到 90 分的开发者。

### RAG 成功的两个前提

高级 RAG 的目标，就是更好地满足这两个条件：

1. **检索**：能找到与用户问题最相关的文档
2. **生成**：LLM 能充分利用检索到的文档，给出充分、准确的回答

高级技巧要么针对检索、要么针对生成，要么同时优化两者。

### 基础 RAG 长什么样？

```python
# LlamaIndex 基础 RAG 流程
documents = SimpleDirectoryReader(input_dir="...").load_data()
index = VectorStoreIndex.from_documents(documents)  # 自动分块、嵌入、建索引
query_engine = index.as_query_engine()
response = query_engine.query("用户的问题")
```

### 针对检索的高级技巧

#### 1. 分块大小优化（Chunk-Size Optimization）

块太大或太小都会影响生成质量。LlamaIndex 提供 `ParamTuner`，可以用网格搜索找到合适的 `chunk_size`、`top_k` 等参数，并结合语义相似度等指标评估。

#### 2. 结构化外部知识（Structured External Knowledge）

当知识库复杂、来源多样时，可以：

- 用文档摘要做高层检索，再定位到具体块
- 用小块做检索，但把关联的大块传给生成阶段（递归检索）

这样检索更精准，生成时又有足够上下文。

#### 3. 查询变换（HyDE 等）

- **HyDE（假设文档嵌入）**：先让 LLM 生成一个「假设的理想答案」，用这个答案的向量去检索，往往比用原始问题检索效果更好
- **多查询**：把一个问题改写成多个不同表述，分别检索后合并结果，提高召回率

#### 4. 融合检索（Fusion Retrieval）

结合向量检索和关键词检索（如 BM25），取长补短。

### 针对生成的高级技巧

#### 1. 信息压缩（Information Compression）

检索到的文档可能包含大量无关内容。可以用 LongLLMLingua 等后处理器，在送入 LLM 前压缩上下文，只保留与问题最相关的部分，减少噪音和 Token 消耗。

#### 2. 重排序（Re-Ranking）

LLM 存在「Lost in the Middle」现象：更容易关注提示的开头和结尾。可以在检索后、生成前，用专门的重排序模型（如 Cohere Rerank）对文档重新排序，把最相关的排在前面。

### 同时优化检索和生成的技巧

#### 1. 生成器增强检索（Generator-Enhanced Retrieval）

让 LLM 在检索前参与进来：根据用户问题，生成更精确的检索指令或子问题，再用这些指令去检索。例如 FLARE 类方法，让 LLM 主动「告诉」检索系统需要什么信息。

#### 2. 迭代式检索-生成（Iterative Retrieval-Generation）

对于复杂问题，可以多轮检索和生成：先检索 → 生成 → 评估答案质量 → 若不满意则调整查询再检索，直到满足条件或达到最大轮数。LlamaIndex 的 `RetryQueryEngine` 就是这种思路。

### RAG 的评估维度

文章引用了 RAG 综述论文中的 7 个评估维度，LlamaIndex 也提供了相应评估工具：

- 检索质量（Relevance）
- 答案忠实度（Faithfulness）：是否基于检索内容，而非幻觉
- 答案相关性（Answer Relevancy）
- 上下文相关性（Context Relevancy）
- 等

建议用 `BatchEvalRunner` 等工具，在代表性问题上系统评估你的 RAG  pipeline。

### 小结：从基础到高级的路径

| 阶段 | 可以做的事 |
|------|------------|
| 基础 RAG | 分块 → 嵌入 → 向量库 → 检索 Top-K → 生成 |
| 检索优化 | 调 chunk_size、用 HyDE/多查询、融合检索、递归检索 |
| 生成优化 | 上下文压缩、重排序 |
| 端到端优化 | 查询改写、迭代检索-生成、系统评估 |

---

## 总结：串联所有知识点

学完以上 6 个资源，你可以建立起这样的 RAG 知识框架：

```
RAG 技术栈
├── 是什么？→ 检索增强生成：让 LLM 在答题前先「翻书」（IBM 视频、RAG from Scratch）
├── 文档怎么处理？→ 分块策略决定检索质量（Pinecone 分块详解）
├── 数据存在哪？→ 向量数据库（ChromaDB 入门、Qdrant 生产）
├── 怎么更好？→ 高级技巧：查询改写、HyDE、重排序、压缩等（LlamaIndex）
└── 怎么实现？→ 从零搭建：索引 → 检索 → 生成（RAG from Scratch）
```

### 关键认知

1. **RAG 解决的是 LLM 的「知识边界」问题**。通过外部知识库 + 检索，让模型能回答私有、最新、可追溯的问题，而不必依赖微调。

2. **分块是 RAG 的「隐藏杠杆」**。块太大检索不准，块太小丢失上下文。没有万能策略，需要结合内容类型、嵌入模型、使用场景反复试验。

3. **向量数据库是 RAG 的基础设施**。学习阶段用 ChromaDB 足够；上生产时考虑 Qdrant 等支持过滤、分布式、高可用的方案。

4. **基础 RAG 只是及格线**。要提升效果，需要系统性地优化检索（查询改写、HyDE、混合检索）和生成（压缩、重排序），并用评估指标驱动迭代。

5. **动手比看书重要**。建议顺序：先看 IBM 视频建立概念 → 跟 RAG from Scratch 跑通流程 → 用 Pinecone 文章调分块 → 选 ChromaDB 或 Qdrant 做项目 → 用 LlamaIndex 文章加高级技巧。

---

**返回** → [阶段 4：RAG 检索增强生成](./index.md)
