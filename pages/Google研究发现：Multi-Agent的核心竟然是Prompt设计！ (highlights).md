title:: Google研究发现：Multi-Agent的核心竟然是Prompt设计！ (highlights)
author:: [[QQ.com]]
full-title:: "Google研究发现：Multi-Agent的核心竟然是Prompt设计！"
category:: #articles
url:: https://mp.weixin.qq.com/s/GDtn1disKmnCPFXzjqzjIw
summary:: Google and Cambridge University discovered that effective prompt design is crucial for multi-agent systems (MAS). They developed a framework called Mass, which optimizes both prompts and topologies, leading to significant performance improvements. This approach shows that simultaneous optimization is more effective than focusing on either aspect alone.
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/AE74ia62XricE533icJ22gK6YUqZPicQiaF4UPVufLKLAIOBLypBj5vTeMOjafPAXKgeaAaI0aIhddyjbC3kibyDznnQ/0?wx_fmt=jpeg)

- Highlights first synced by [[Readwise]] [[Jun 15th, 2025]]
	- 为了自动化整个设计过程，Google&剑桥大学首先对设计空间进行了深入分析，旨在了解构建有效MAS的因素。发现：提示设计对下游性能有显著影响，而有效的拓扑结构只占整个搜索空间的一小部分。 ([View Highlight](https://read.readwise.io/read/01jxsza1f8836kssvdmwnrcvja))
- New highlights added [[Jun 17th, 2025]] at 8:32 PM
	- 在数学问题上，Gemini 1.5 Pro 对比仅使用自我一致性（SC）、自我细化（reflect）和多智能体辩论（debate）进行扩展的智能体，展示了每个问题的提示优化智能体的准确率与总标记数的关系。误差条表示1个标准差。我们表明，通过更有效的提示，利用更多的计算资源可以获得更高的准确率。 ([View Highlight](https://read.readwise.io/read/01jxyvsgxq3edv4r3vmbhgcgm6))
	- 实验使用了Gemini 1.5 Pro和Flash模型，并与多种现有方法进行了比较，包括链式思考（CoT）、自我一致性（SC）、自我细化（Self-Refine）、多智能体辩论（Multi-Agent Debate）、ADAS和AFlow。 ([View Highlight](https://read.readwise.io/read/01jxyvhgybkv91r02cb3fgs43d))
	- •   **优化阶段的重要性**：通过分阶段优化，Mass在每个阶段都取得了性能提升，证明了从局部到全局优化的必要性。
	    
	  •   **提示和拓扑结构的协同优化**：Mass通过同时优化提示和拓扑结构，实现了比单独优化更好的性能。
	    
	  •   **成本效益**：Mass在优化过程中表现出稳定且有效的性能提升，与现有自动设计方法相比，具有更高的样本效率和成本效益。 ([View Highlight](https://read.readwise.io/read/01jxyvj15egjg7wf3akvdw1fjw))