title:: Rag 2.0性能提升：优化索引与召回机制的策略与实践 (highlights)
author:: [[知乎专栏]]
full-title:: "Rag 2.0性能提升：优化索引与召回机制的策略与实践"
category:: #articles
url:: https://zhuanlan.zhihu.com/p/5174032253
summary:: The text discusses the challenges and improvements in the RAG 2.0 engine, focusing on optimizing indexing and recall mechanisms. Key topics include effective chunking, the development of next-generation RAG architectures, and strategies to enhance data quality and retrieval accuracy. The presentation highlights the integration of advanced models and techniques to tackle issues in document retrieval and multi-modal data processing.
![](https://readwise-assets.s3.amazonaws.com/static/images/article3.5c705a01b476.png)

- Highlights first synced by [[Readwise]] [[Apr 15th, 2025]]
	- Graph embedding 其实也可以去做更多的改进。一种简单的例子是以企业内部问答系统去对图神经网络做一个近似，可以把 node embedding 质量变得更高。如果直接生成 embedding 其实相当于对知识图谱的图做了一个遍历，类似于 PageRank，那为什么做 PageRank 可以把知识图谱的召回做得比较符合我们答案的诉求呢？是因为这符合我们大脑思维的过程，我们在联想的时候，大脑内部类似于走了一个随机游走的过程，这个过程我们通过一个叫做 node2Vect 的算法把它变成 graph embedding。因此有了知识图谱，再通过 graph embedding 去做召回，就可以很好地处理多跳问答或者比较宏观的问答方式。 ([View Highlight](https://read.readwise.io/read/01jrkz9fn9g8342g67btqp14k3))
	- A1: Sparse Vector 用的是 BGE-M3。BGE-M3 是目前我们使用比较多的一种模型，它可以输出向量、稀疏向量和张量。但张量输出存在一些问题，其张量达到了 1024 维，其实是完全没有必要的，对于张量来说，维度达到 128 就已经足够了。BERT 的话，我们现在用的就是 ColBERT 模型，大家可以去 HuggingFace 下载。Jina-ColBERTv2 是 Jina 大概两三周前刚刚开源出来的模型，大家可以去尝试一下，主要是其中文指标，比 BGE-M3 还是要低不少，该模型有很大的改进余地。 ([View Highlight](https://read.readwise.io/read/01jrqmyej4msd86bb9h94g9kzx))
	- 表格我们现在其实是存两块：HTML 和向量。由于 HTML 用的是全文索引，所以表格更多的是以全文索引的方式去做召回，向量只是一种辅助。在企业里面如果不是用于多语言场景，基本上全文索引的权重会更高。对于“大模型对于表格的数字不是太敏感，容易产生幻觉”，我们目前还是必须得依赖大模型，因为我们在一开始的时候是把每个单元格变成一句话，这在表格绝对准确的情况下的会更好。一旦单元格错了一个，可能就会导致整个的关系对齐都会出现错误。所以有那么一两个单元格识别的不够准，仍然有机会让大模型去做挽回。未来希望表格模型能够变得尽可能达到 100% 准确，我们可能就又会回到把它变成一句话的方式。 ([View Highlight](https://read.readwise.io/read/01jrqn584b2cvgzbdh4zy5bsd5))
	- ![](https://pic1.zhimg.com/v2-9cf781aa94a35d3ba02f0f4123558e0c_1440w.jpg) ([View Highlight](https://read.readwise.io/read/01jrqzba8e468xpqxcxq4c2dh5))
	- ![](https://pic1.zhimg.com/v2-4b91fb410257865312c17f7c9d7b1c90_1440w.jpg) ([View Highlight](https://read.readwise.io/read/01jrn043zfrjt5bwrhvsffkjgk))
	- 我们现在正在训练的模型正是采用这种完全基于 transformer 的架构，无论是表格还是流程图、饼图、柱状图都可以处理，因为这是一种通用的方案，解决图片进-文字出、encoder 进-decoder 出的问题
	  
	  。具体过程为，第一步用 VAE（变分自动编码器）的方式做特征抽取，先用 CNN 编码器把表格的图片编码，然后再用 CNN 解码器还原，让我们的解码器和编码器能得到的图片尽可能跟原始图片一样，我们就得到了中间的 Code Book（码本）。这其实就相当于 image patch 的一个 embedding，能够非常真实地还原表格场景中 embedding 的表示，这个 Code Book 是非常重要的。
	  
	  第二步是训练 Transformer Encoder。Encoder 同样是 image 进，并且要让 Encoder 的输出尽可能去拟合上面的 embedding。第三步是用 Encoder 和 Decoder 一起训练。Decoder 输出最终的一个 HTML 文本。这种结构跟大模型是有一定相似性的，只不过大模型如 GPT 都是 Decoder only。但是我们只要做多模态模型就必须是 Encoder Decoder 这种架构，以得到一个统一的图像转文本的方案。
	  
	  虽然模型仍在训练中，但已训练出来的结果显示表格识别的效果非常好，比之前用 CNN 的鲁棒性能要好很多。因为表格识别模型基于 Transformer 架构，这种模型的训练都有个比较高的门槛就是训练数据的来源。我们现在的做法基本上都是用程序来生成，尽可能去覆盖更多的场景。比如哪些用户的表格做的不够好，我们就专门针对这种场景去做模拟生成相应的图片，然后拿这种图片去不断迭代模型，最终形成一个数据飞轮，使模型迭代效果、泛化能力越来越好。 ([View Highlight](https://read.readwise.io/read/01jrn07j2htppwrffqpe7sgn07))
	- 目前用于 RAG 的数据库分为三类：
	  
	  •   第一类是传统型数据库。这种类型的数据库只要增加了向量能力，理论上就可以用于 RAG。像 PostgreSQL 有个著名的插件 PG Vector 就是用来支持向量存取的，Snowflake 是一个数仓，同时也具备向量的能力。
	  •   第二类是典型的向量数据库，如 Pinecone、Qdrant、Weaviate。
	  •   第三类是具备全文搜索+向量能力的数据库，比如 ROCKSET、LanceDB、Elasticsearch。
	  
	  在企业级场景中，全文搜索是一项必备能力，目前知名的具备全文搜索和向量能力的数据库就是以上这些。LanceDB是最近北美孵化出来的一个数据库，采用了知名的 Tantivy 库做全文索引。ROCKSET 是 Open AI 在今年六月份收购的一家数据库公司，它是一个索引型数据库，对每一列都建了全文索引，所以一开始就是去取代 Elasticsearch 的，不过后来因为 RAG 的流行，它又增加了向量索引，因此具备了两路混合搜索，以保证更好的召回结果。我们也正在将 Infinity 这个数据库加到 RAGFlow 中。因为 RAGFlow现在用的是 Elasticsearch，替换成 Infinity 还需要一点时间。 ([View Highlight](https://read.readwise.io/read/01jrn0gnvnd1x18bv6ww6x3h29))
	- 接下来讨论一些技术问题，第一个问题就是我们现在已经有向量、稀疏向量、张量等搜索方式，那混合搜索、多路召回还有意义吗？我们使用 MLDR 数据集做了一个评测。MLDR 是一个长文档数据集，我们用自己的数据库跑出了如上图所示的指标，图中纵坐标为 nDCG@10，对每个结果的位置都要去对最终的结果做一个加权的得分。所以本来是排在第一位的，放到第二位也会对这个分数产生影响。图中颜色最浅的这三路就是我们用三种方式分别去搜索，BM25 是全文搜索，Dense 是向量搜索，Sparse 是稀疏向量。可以看到如果只用向量的话，最低的只有 49 分。中间颜色深一点的是两路召回，就是把稀疏向量和全文搜索两两组合，再加上一个比较 basic 的融合排序，即两路召回加 RRF 得到一个混合搜索，结果确实比单路搜索要好一些。倒数第二个是把三路搜索混在一起，再加 RRF，它的得分更高。这个结果来自今年四月份 IBM Research 苏黎世的研究员发表的一篇名为《BlendedRAG: Improving RAG》的论文（论文地址：[https://arxiv.org/pdf/2404.07220](https://link.zhihu.com/?target=https%3A//arxiv.org/pdf/2404.07220)），其结论就是三路召回效果最好。我们通过我们的数据库也复现出了这个结论。图中最右边，又有一个比较大的提升，因为我们做了个张量，将张量以 [ColBERT](https://zhida.zhihu.com/search?content_id=250057218&content_type=Article&match_order=1&q=ColBERT&zd_token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJ6aGlkYV9zZXJ2ZXIiLCJleHAiOjE3NDQ1OTk1MzUsInEiOiJDb2xCRVJUIiwiemhpZGFfc291cmNlIjoiZW50aXR5IiwiY29udGVudF9pZCI6MjUwMDU3MjE4LCJjb250ZW50X3R5cGUiOiJBcnRpY2xlIiwibWF0Y2hfb3JkZXIiOjEsInpkX3Rva2VuIjpudWxsfQ.6N2kZA2HGBMujUhUafiSUjOvnMDPiNtAr3sQx_hQ0Xo&zhida_source=entity) 的形式放进来，使最终的召回效果有了更大的提升。 ([View Highlight](https://read.readwise.io/read/01jrn0jm5361rj7vx6a229rxbf)) #[[todo]]
		- 💡: 什么是多路召回
	- **延迟交互是 RAG 的未来**
	  
	  ![](https://pic1.zhimg.com/v2-e2f5f24eb9d87be03076251eeff82920_1440w.jpg)
	  
	  根据我们的观察，延迟交互编码并不是交叉编码的一个 trade off，它甚至可以做得比交叉编码更好。上图中第一个图是来自 JaColBERT的数据，这是今年八月份北美一家叫做昂斯利亚的公司训练出来的专门针对日本的ColBERT的模型，可以看到在 MIRACL 这个数据集上，它的表现甚至比 BGE-M3 还要好。所以基于这种延迟交互的模型，可能会带来更高的精度。因此我们认为张量是 RAG 的未来发展方向。 ([View Highlight](https://read.readwise.io/read/01jrpyt3489rvxaj92z7ghnb5w))
	- 外一个模型是 answerai。它把 ColBERT 的参数压缩到只有 3300 万，但是获得的效果比 BGE 一亿多参数的模型更好。而且它的每个 token 只有 96 位，如果做二进制量化之后，那么每个 token 只有 12 个字节，这样空间浪费会大大减少。相信在未来会有更多这种 ColBERT 模型出现，特别是用于中文的 ColBERT模型，来保证召回的效果。
	  
	  **04** ([View Highlight](https://read.readwise.io/read/01jrpywt75z2hr5j0dn5tby1r4))
	- 另外一个模型是 answerai。它把 ColBERT 的参数压缩到只有 3300 万，但是获得的效果比 BGE 一亿多参数的模型更好。而且它的每个 token 只有 96 位，如果做二进制量化之后，那么每个 token 只有 12 个字节，这样空间浪费会大大减少。相信在未来会有更多这种 ColBERT 模型出现，特别是用于中文的 ColBERT模型，来保证召回的效果。 ([View Highlight](https://read.readwise.io/read/01jrpyxefd830hdnznw95engmt))
	- 亿多参数的模型更好。而且它的每个 token 只有 96 位，如果做二进制量化之后，那么每个 token 只有 12 个字节，这样空间浪费会大大减少。相信在未来会有更多这种 ColBERT 模型出现，特别是用于中文的 Col ([View Highlight](https://read.readwise.io/read/01jrpyxekab5fc010fkn3wesdx))
	- 第一种预处理方法称为 RAPTOR，首先是对文档去做聚类，做好聚类之后生成摘要，连同这些信息一起作为 chunks 送到 RAG 体系里面去，在搜索的时候我们就可以搜索到这个聚类后的信息。整个文档跨 chunks 之间具备语义信息，所以基于 RAPTOR 可以去解决多跳问答或者比较宏观的问答。 ([View Highlight](https://read.readwise.io/read/01jrpz2xaap2cfj9vpy2pqp2rr))