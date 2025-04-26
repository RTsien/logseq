title:: 如何用 Cursor 在一个大项目上更高效地工作？ (highlights)
author:: [[乔梁]]
full-title:: "如何用 Cursor 在一个大项目上更高效地工作？"
category:: #articles
url:: https://mp.weixin.qq.com/s/6I3b1pzMFVcuSlh7wuCkCw
summary:: 本文介绍了如何利用Cursor在大项目中提升效率。重点包括良好的文档管理、使用Notepad功能、以及定期对代码库进行重新索引。通过这些方法，可以更好地管理项目进度和技术实现，从而有效利用AI助手。
![](https://mmbiz.qpic.cn/mmbiz_jpg/YHiaESnpiaFDA30rPRxd4rLInMYIUMuaibzMNTBL3XzRjHvOaQK3OkZdRpTFtzOEQFGOoNb1AmqZXJTnNpvQFMemQ/0?wx_fmt=jpeg)

- Highlights first synced by [[Readwise]] [[Apr 11th, 2025]]
	- 2使用 Notepad
	  
	  Notepad 功能目前 (205.04) 还是 beta 阶段，所以你需要在 Cursor 的配置中打开。
	  
	  ![](https://mmbiz.qpic.cn/mmbiz_png/YHiaESnpiaFDA30rPRxd4rLInMYIUMuaibzzGq0zwTH7KgXbzRINRoUjPeEblnSMEeP2pXTZiaiaZQmIAv9DSRshDqg/640?wx_fmt=png)
	  
	  Notepad
	  
	  这样，你就可以在左面板中增加下面两个 Notepad 了（当然，你可以添加你自己的 Notepad）。
	  
	  1.  'sync': 停下来整理思路，检查是否已更新 @project.md 和 @progress.md（如果需要的话），确保你知道我们目前进展到哪里了，然后我们再进入下一步。先不要开始编码。
	  2.  'Get to up': 检查 @project.md 和 @progress.md 并充分理解我们的项目和进展 - 基于此建议下一步行动。先不要开始编码
	  
	  然后我会定期在编辑器中输入 @sync 作为提示。有时在 `New Chat`时，也会这样做。 ([View Highlight](https://read.readwise.io/read/01jrdjfvzpb1g1bny9apeaa2y0))
	- 3使用项目里程碑文档管理项目进度
	  
	  创建一个 `Project_milestones.md` 文件，并在 `.cursorrules` 文件中引用它。
	  
	  尽可能详细地告诉Cursor你的项目范围和需求，让它生成一个项目文件。在每个会话结束时（或在会话期间），让Cursor更新该文件，标记已完成里程碑和学到的经验。 ([View Highlight](https://read.readwise.io/read/01jrdjfd64vxsgf8vjceetafn5))
	- 4使用技术说明文档来记录相关的实现内容。
	  
	  创建一个 `Documentation.md` 文件，并在 `.cursorrules`文件中引用它。
	  
	  该文档中应该记录本项目的技术架构与实现。你要确保让 Cursor 定期更新（或让Cursor自行更新）新加入或修改过的函数、架构等。 ([View Highlight](https://read.readwise.io/read/01jrdjf72gq3vqh4d1da7gyq2d))
	- 6循序渐进地工作。
	  
	  Cursor 目前还只算是掌握大量知识的实习生。不要要求 Cursor 做巨大的功能更新， 例如“给我写 XX APP”。而是指导 Cursor 使用project.md。或者你直接要求 Cursor，"现在，让我们开始下一个功能， 1.2 blahblah "，它总能遵循里程碑顺序进行。
	  
	  7做好版本管理
	  
	  一旦达到了一个成功点，要提交代码。你可以让它总结最近的修改。
	  
	  8你还可以做这些
	  
	  •   在项目根目录创建 this.log 文件，并始终将调试日志复制/粘贴到该文件中，然后在聊天中引用它。如果你直接将错误日志粘贴到聊天对话中，很快 Cursor 的上下文就会被破坏了。
	    
	  
	  •   当你想要添加或修改一个复杂功能时，与其自己分解任务并只给出下一步要做什么，不如先向它完整地解释整个功能，让它在不涉及代码的情况下制定一个高层次的计划，然后对计划提出改进建议，最后再让它开始实施。
	    
	  
	  •   增加一个 code rule， 让代码文件不要超过 500行。 ([View Highlight](https://read.readwise.io/read/01jrdjcpkza0as4g9vj03pw6p2))