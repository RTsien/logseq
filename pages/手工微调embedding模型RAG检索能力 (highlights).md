title:: 手工微调embedding模型RAG检索能力 (highlights)
author:: [[yuanwai]]
full-title:: "手工微调embedding模型RAG检索能力"
category:: #articles
url:: https://luxiangdong.com/2023/09/27/ftembedding/
summary:: 这篇文章介绍了如何微调开源的embedding模型big-large-en，以提高其在RAG（检索增强生成）应用中的检索能力。作者使用LlamaIndex工具生成训练和评估数据集，并通过微调模型来评估其性能提升。结果显示，微调后的模型在检索准确率上比基本模型提高了1-6%，与OpenAI模型相比，性能损失仅为4.85%。
![](https://luxiangdong.com/favicon.ico)

- Highlights first synced by [[Readwise]] [[Apr 25th, 2025]]
	- 微调Embedding模型似乎涉及到很多问题。幸运的是，LlamaIndex（我个人感觉LlamaIndex目前的发展可能会在RAG方面打败LangChain）在最近的0.8.21版本中引入以下关键类/函数，使得微调Embedding模型变得超级简单:
	  
	  •   `SentenceTransformersFinetuneEngine`
	  •   `generate_qa_embedding_pairs`
	  •   `EmbeddingQAFinetuneDataset` ([View Highlight](https://read.readwise.io/read/01js29k2apb7cfefhczjwy3kf0))