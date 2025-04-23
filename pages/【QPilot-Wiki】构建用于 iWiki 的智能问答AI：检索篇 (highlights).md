title:: 【QPilot-Wiki】构建用于 iWiki 的智能问答AI：检索篇 (highlights)
author:: [[woa.com]]
full-title:: "【QPilot-Wiki】构建用于 iWiki 的智能问答AI：检索篇"
category:: #articles
url:: https://km.woa.com/articles/show/581776
summary:: The article discusses the development of an intelligent Q&A AI for iWiki, focusing on data retrieval techniques. It highlights the challenges of processing iWiki's rich text format and the use of embedding search methods to improve answer accuracy. The QPilot-wiki model has achieved a recall rate of 95% for frequently asked questions through various optimizations.
![](https://readwise-assets.s3.amazonaws.com/static/images/article4.6bc1851654a0.png)

- Highlights first synced by [[Readwise]] [[Apr 11th, 2025]]
	- 这里比较难处理的部分是表格，有些表格有合并单元格，是不被 MD 格式支持，如果转为 HTML 则 token 数过高。这里的处理方式是把合并的单元格展开；
	  
	  此外，附件、图片、链接、腾讯文档等由于性能和API能力关系暂时不索引，因此在回答得时候会作为链接附上 ([View Highlight](https://read.readwise.io/read/01jrek9rafqzpkrm3qb2gjehj5))
	- **数据检索**
	  
	  数据准备好了之后，由于LLM输入大小限制（LLaMa 系都是2k），所以在回答用户问题的时候，需要把问题相关的知识先取出来，然后让LLM回答。
	  
	  这就需要信息检索（information retrieval，IR）技术，常见做法有：
	  
	  •   基于文本搜索
	  •   基于图数据库
	  •   基于词嵌入（Embedding）搜索
	  
	  对比见：[Search: Query Matching via Lexical, Graph, and Embedding Methods](https://eugeneyan.com/writing/search-query-matching/)
	  
	  其中基于 embedding 搜索能够捕捉词汇和语义的相似性，对于同义词、近义词和语义相似的表达方式具有较好的识别能力。对于错别字、拼写错误等情况，embedding 搜索具有一定的容错性，因此比传统文本搜索效果更好。 ([View Highlight](https://read.readwise.io/read/01jrekc19jx69g52k0ej4yjzdr))