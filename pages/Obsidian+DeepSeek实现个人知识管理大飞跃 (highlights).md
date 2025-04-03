title:: Obsidian+DeepSeek实现个人知识管理大飞跃 (highlights)
author:: [[知乎专栏]]
full-title:: "Obsidian+DeepSeek实现个人知识管理大飞跃"
category:: #articles
url:: https://zhuanlan.zhihu.com/p/22007694129
summary:: The author discusses how to use Obsidian and DeepSeek to create a personal knowledge management system that is efficient and AI-powered. This setup allows for smarter knowledge extraction, improved writing assistance, and better connections between notes. The article provides a step-by-step guide for those interested in building their knowledge base using these tools.
![](https://readwise-assets.s3.amazonaws.com/static/images/article4.6bc1851654a0.png)

- Highlights first synced by [[Readwise]] [[Mar 17th, 2025]]
	- 至于为什么需要[LM Studio](https://zhida.zhihu.com/search?content_id=253423588&content_type=Article&match_order=1&q=LM+Studio&zd_token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJ6aGlkYV9zZXJ2ZXIiLCJleHAiOjE3NDIwNDAxNTEsInEiOiJMTSBTdHVkaW8iLCJ6aGlkYV9zb3VyY2UiOiJlbnRpdHkiLCJjb250ZW50X2lkIjoyNTM0MjM1ODgsImNvbnRlbnRfdHlwZSI6IkFydGljbGUiLCJtYXRjaF9vcmRlciI6MSwiemRfdG9rZW4iOm51bGx9.9g7-DAsQP54c6qOj-eatOArHLvpUWDiM6JyEjx5CrMs&zhida_source=entity)，是因为大模型本身只提供了问答的接口。必须对笔记内容进行向量化处理才能跟大模型进行知识库的交互。我没试过[Embedding模型](https://zhida.zhihu.com/search?content_id=253423588&content_type=Article&match_order=1&q=Embedding%E6%A8%A1%E5%9E%8B&zd_token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJ6aGlkYV9zZXJ2ZXIiLCJleHAiOjE3NDIwNDAxNTEsInEiOiJFbWJlZGRpbmfmqKHlnosiLCJ6aGlkYV9zb3VyY2UiOiJlbnRpdHkiLCJjb250ZW50X2lkIjoyNTM0MjM1ODgsImNvbnRlbnRfdHlwZSI6IkFydGljbGUiLCJtYXRjaF9vcmRlciI6MSwiemRfdG9rZW4iOm51bGx9.TIw171lKZ9htmjnH6MsBTqTyH0FXJZsDPaNnxn_06X8&zhida_source=entity)是否也可以用API接口，因为我的笔记库本身很大，上千条笔记，内容太多了，感觉用API会很慢。Embedding模型本身不大，一般电脑都能跑起来，也没有显卡限制，用起来还是很方便的。第一次用Embedding模型建索引会有些慢，但是索引结果会存在[Copilot插件](https://zhida.zhihu.com/search?content_id=253423588&content_type=Article&match_order=1&q=Copilot%E6%8F%92%E4%BB%B6&zd_token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJ6aGlkYV9zZXJ2ZXIiLCJleHAiOjE3NDIwNDAxNTEsInEiOiJDb3BpbG905o-S5Lu2IiwiemhpZGFfc291cmNlIjoiZW50aXR5IiwiY29udGVudF9pZCI6MjUzNDIzNTg4LCJjb250ZW50X3R5cGUiOiJBcnRpY2xlIiwibWF0Y2hfb3JkZXIiOjEsInpkX3Rva2VuIjpudWxsfQ.NTU2O_tNhx0mPEjA7jOOtCPwbJqla-0f_2AL3jfwkcg&zhida_source=entity)的缓存文件里，后续每次只会对有变化的笔记重新索引，系统资源占用也很小。 ([View Highlight](https://read.readwise.io/read/01jp8620kmaw94gbeyb1y2c3pv))
		- 💡: cursor的codebase是不是也先通过embedding api完成了索引？
		  
		  一下是claude 3.7的回答
		  
		  Cursor的工作原理包括:
		  使用embedding API将代码库中的代码片段转换为向量表示
		  建立这些向量的索引，以便快速检索
		  当用户进行搜索或需要上下文理解时，可以通过向量相似性快速找到相关代码
		  https://monica.im/share/chat?shareId=JBhGBNfaIwi1cMMh