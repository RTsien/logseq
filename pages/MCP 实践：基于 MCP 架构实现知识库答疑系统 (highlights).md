title:: MCP 实践：基于 MCP 架构实现知识库答疑系统 (highlights)
author:: [[木洛]]
full-title:: "MCP 实践：基于 MCP 架构实现知识库答疑系统"
category:: #articles
url:: https://mp.weixin.qq.com/s/ETmbEAE7lNligcM_A_GF8A
summary:: This article discusses the development of AI Agents and showcases how to create a Q&A system using the Model Context Protocol (MCP) architecture with a private knowledge base. It outlines the process of building and retrieving knowledge, highlighting optimizations like FAQ extraction and mixed retrieval methods. The author emphasizes the rapid evolution of AI applications and the potential impact of MCP as a standard in this field.
![](https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naLMrQjmqJia3ibOfKdl9bx9AjLDCLZHAh4gmj55iamibyQsJvGzeZtNR6tZCqEHpoSRRYictpVIqickJrAA/0?wx_fmt=jpeg)

- Highlights first synced by [[Readwise]] [[Apr 25th, 2025]]
	- 整体流程设计
	  
	  ![](https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naLMrQjmqJia3ibOfKdl9bx9AjgibW1jicvOEmickAUE1hCpwjOibY7tdwnTd3WKHvxcPrM6QdHfibL4PiaD5Q/640?wx_fmt=other&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
	  
	  主要分为两部分：知识库构建和检索。
	  
	  1.知识库构建
	  
	  a.文本切段：对文本进行切段，切段后的内容需要保证文本完整性以及语义完整性。
	  
	  b.提取 FAQ：根据文本内容提取 FAQ，作为知识库检索的一个补充，以提升检索效果。
	  
	  c.导入知识库：将文本和 FAQ 导入知识库，并进行 Embedding 后导入向量。
	  
	  2.知识检索（RAG）
	  
	  a.问题拆解：对输入问题进行拆解和重写，拆解为更原子的子问题。
	  
	  b.检索：针对每个子问题分别检索相关文本和 FAQ，针对文本采取向量检索，针对 FAQ 采取全文和向量混合检索。
	  
	  c.知识库内容筛选：针对检索出来的内容进行筛选，保留与问题最相关的内容进行参考回答。
	  
	  相比传统的 Naive RAG，在知识库构建和检索分别做了一些常见的优化，包括 Chunk 切分优化、提取 FAQ、Query Rewrite、混合检索等。 ([View Highlight](https://read.readwise.io/read/01jsdqvt01f3apb7pqcs60k0ax))
	- Agent 架构
	  
	  ![](https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naLMrQjmqJia3ibOfKdl9bx9Aj01ze4JuqSx1a8zOoj9V8gjx49zGialBlZS9absgFF8tiaLH4NyYczeKQ/640?wx_fmt=other&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
	  
	  整体架构分为三个部分：
	  
	  1.知识库：内部包含 Knowledge Store 和 FAQ Store，分别存储文本内容和 FAQ 内容，支持向量和全文的混合检索。
	  
	  2.MCP Server：提供对 Knowledge Store 和 FAQ Store 的读写操作，总共提供 4 个 Tools。
	  
	  3.功能实现部分：完全通过 Prompt + LLM 来实现对知识库的导入、检索和问答这几个功能。 ([View Highlight](https://read.readwise.io/read/01jsdqw385n2k2q9ntme69gszv))
	- **MCP Server**
	  
	  实现了 4 个 Tools（具体注册代码可参考 TablestoreMcp），相关描述如下：
	  
	  ![](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naLMrQjmqJia3ibOfKdl9bx9AjIWrb0pTmKpRXCkZnBFpSTV1abhQkOkIHvR7JW3EeXCuYCqIxia3kS5A/640?wx_fmt=png&from=appmsg)
	  
	  ![](https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naLMrQjmqJia3ibOfKdl9bx9AjvaR35UVS4jvUiasEKkCdK8GkoVFRnapvk0XLgEbK6bEYL0EWLnOLicCA/640?wx_fmt=jpeg) ([View Highlight](https://read.readwise.io/read/01jsdqxfa7c48sj6prep6kqewn))
	- **知识库构建**
	  
	  1、对文本进行切段并提取 FAQ
	  
	  完全通过提示词来完成，可根据自己的要求进行调优。
	  
	  ![](https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naLMrQjmqJia3ibOfKdl9bx9Ajiaib7HvmGk8VuiaSC4Wczmn124T5GbR9iatSkzBHdoA271okTQ8L0fDHLg/640?wx_fmt=jpeg&from=appmsg)
	  
	  ![](https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naLMrQjmqJia3ibOfKdl9bx9AjVvHDbanso8MetQ4BTjDpicxYqtjlPo2Ob8ypN7hRKHfr4oibH3hI4IZA/640?wx_fmt=jpeg&from=appmsg)
	  
	  ![](https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naLMrQjmqJia3ibOfKdl9bx9Ajqic85ia6W7ex6BGCj0iabZBUBMMtGfZ8xa35cEiagjxQiaRXhsibwCibCwK4Q/640?wx_fmt=jpeg&from=appmsg)
	  
	  以上是一个示例，可以看到通过大模型能比较准确的对文本进行切段并提取 FAQ。这种方式的优势是切段的文本能保证完整性以及语义一致性，能够比较灵活的对格式做一些处理。提取的 FAQ 很全面，对于简单问题的问答通过直接搜索 FAQ 是最准确直接的。最大的缺点就是执行比较慢并且成本较高，一次会消耗大量的 Token，不过好在是一次性的投入。
	  
	  2、写入知识库和 FAQ 库
	  
	  这一步也是通过提示词来完成，基于 MCP 架构可以非常简单的实现，样例如下：
	  
	  ![](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naLMrQjmqJia3ibOfKdl9bx9AjyoG1DFdtHAhLQ6LnokYlanBJ249o24QrAd2hH1hW3RQGxxMyqHjyTA/640?wx_fmt=png&from=appmsg)
	  
	  **知识库检索**
	  
	  同样这一步也是通过提示词加 MCP 来实现，非常的简单，样例如下：
	  
	  ![](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naLMrQjmqJia3ibOfKdl9bx9Ajpj9BOwGicqoub4YIOmQZFoA5daCicVxicGkCQB2eh8jReSuTMsvMGUsxA/640?wx_fmt=png&from=appmsg)
	  
	  通过提示词描述实现了一个稍微复杂点的检索：
	  
	  1.先对问题进行拆解，拆解为更原子的子问题。
	  
	  2.每个子问题分别检索知识库和 FAQ，检索结果汇总后筛选留下与问题最相关的内容。
	  
	  3.按照格式返回结果。
	  
	  **知识库问答**
	  
	  直接看下提示词和效果
	  
	  ![](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naLMrQjmqJia3ibOfKdl9bx9AjSbs84ru1wiccfoIDHqNcQd2VO56iaV8oJRwXY73lFJC3ZUl0y1U2ia77g/640?wx_fmt=png&from=appmsg)
	  
	  从 MCP Server 的 Log 内可以看到自动调用了知识库和 FAQ 的检索工具，并能根据之前导入的内容进行回答。 ([View Highlight](https://read.readwise.io/read/01jsdr196r6pxhmwkcfd9s1pse))