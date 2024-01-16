title:: 纯前端技术选型 (highlights)
author:: [[woa.com]]
full-title:: "纯前端技术选型"
category:: #articles
url:: https://km.woa.com/articles/show/595986
![](https://readwise-assets.s3.amazonaws.com/static/images/article0.00998d930354.png)

- Highlights first synced by [[Readwise]] [[Jan 13th, 2024]]
	- 这种增强，有点“先搜索，把搜索结果全部丢给LLM，让它总结一下”的意味。其中，向量化一方面是实现检索本身，另一方面是减少token。本质上，我觉得不一定要我们自己实现向量化，我们甚至可以直接依赖百度、谷歌，把前10篇文章的内容抓回来，再整理成prompt给LLM，也可以得到相同的效果。这里似乎只需要利用大模型可以归纳总结的能力即可，当然，LLM能理解人类语言的优势在检索时仍然有帮助。 ([View Highlight](https://read.readwise.io/read/01hkydr918vw7ajj88jzsn6x4b))
		- 💡: RAG相当于先搜索（检索）出相关信息，然后作为提示词的一部份交给LLM归纳