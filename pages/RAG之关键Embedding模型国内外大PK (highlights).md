title:: RAG之关键Embedding模型国内外大PK (highlights)
author:: [[飘逝的风]]
full-title:: "RAG之关键Embedding模型国内外大PK"
category:: #articles
url:: https://gameapp.club/post/2025-04-02-embedding-compare/
summary:: 虽然大模型支持的上下文是越来越大，但不论出于知识库过大还是基于安全考虑，我们还是希望向模型提供适当的上下文即可。这其中选择合适的embedding模型就至关重要了。如何才能找到效果更好的embedding型呢，希望本文能提供一些参考。

背景
我们不能为技术而技术，最好是解决某项具体问题而进行探索。我为何想去了解embedding这块呢？缘于最近MCP比较火，而我工作中经常需要分析一些仓库的提交历史，以发现某些内容的引入或修改历史，即我想和git历史进行交谈。虽然有时咱传统方式也能做，但写个MCP可以用自然语言获得诸如：

XX玩法是谁负责的
A最近开发的了哪些内容
最近一个月主要有哪些功能在开发
今年3月份有哪些功能
某个文件最近有哪些修改

这一些问题的答案那自然是极好的。这些信息或许可基于git log等进一步检索，而我们一个大项目是由几十个小仓组成的，难度就上升了一层。不过完整的解决方案已经开发得差不多了，今天就先聊一下如何解决第一个挑战，embedding！我计划了一场PK赛，看看哪个模型更适合我的场景。

先叠一层甲，我本人非AI领域人员，基于爱好和专用场景测试，受于个人知识限制，可能存在理解偏差，欢迎指正。

国内外模型介绍
什么是embedding呢？wikipedia的描述比较抽象，以下是腾讯混元T1的解释：

Embedding模型是一种将高维数据（如文本、图像）映射到低维向量空间的技术，通过保留原始数据的语义和特征信息，实现高效计算与相似性分析。其核心原理是通过神经网络训练，将相似的数据点映射到向量空间中的相近位置，例如"猫"和"狗"的向量比"猫"和"苹果"的更接近，从而捕捉语义关联。

在huggingface上有一个排行榜，可以查看不同模型的效果。用于了解有哪些模型还不错，但我们具体使用上还是实测可能更靠谱。
我计划选择免费开源的一些模型，同时也测试一些闭源模型看其提升有多大，是否值得咱付费使用。而这个测试场景，大概有如下几步：

由AI生成一些git commit message。
基于这些message交给待测试的各个embedding模型来向量化。
通过输入Query问题进行相似度(余弦相似度)检索，获得Top5的commit message。
交给AI对各个embedding模型进行打分（有点重复工作量，我们看看AI表现），看Query出的质量如何？

实测上有一些意想不到的结果呢，让我们拭目以待。
开源embedding模型介绍
在网上查看了一些资料后，我选择了如下几个被推荐较多的模型用于后续测试。



模型名称
描述
维度
最大token
支持语言




text-embedding-gte-large-zh
GTE大型中文嵌入模型（本地）
1024
512
中文


text-embedding-bge-large-zh-v1.5
百度开源的中英双语大型嵌入模型（本地）
1024
512
中文、英文


text-embedding-m3e-base
M3E基础嵌入模型（本地）
768
512
中文、英文


text-embedding-granite-embedding-278m-multilingual
Granite多语言嵌入模型（本地）
768
512
多语言（英文、德文、西班牙文、法文、日文、葡萄牙文、阿拉伯文、捷克文、意大利文、韩文、荷兰文、中文等）


text-embedding-multilingual-e5-large-instruct
E5大型多语言嵌入模型
1024
512
多语言



原本jina-embeddings系列模型也想一并参赛的，无奈在LM Studio中支持得不太好，可能缘分未到，暂时跳过。若有朋友有使用经验，不妨留言分享一下实际效果。
闭源大厂embedding模型介绍
以OpenAI为首的如text-embedding-3系列，以及国内各个大厂BAT以及字节等都有自己的embedding模型都获得了参赛资格。这取决于我之前在OneAPI提到过收集的模型提供商了，只要他们有embedding模型，都跃跃欲试进组PK。



模型名称
描述
维度
最大词元数
支持语言




text-embedding-3-large
OpenAI第三代大型嵌入模型
3072
8191
多语言


hunyuan-embedding
腾讯混元嵌入模型
1024
1024
中文、英文


doubao-embedding-large-text-240915
豆包嵌入模型
1024
4096
中文、英文


Baichuan-Text-Embedding
百川嵌入模型
1024
512
中文、英文


text-embedding-v3
通义千问嵌入...
![](https://img.gameapp.club/images/2025/04/embedding-logo.png)

- Highlights first synced by [[Readwise]] [[Apr 25th, 2025]]
	- text-embedding-multilingual-e5-large-instruct ([View Highlight](https://read.readwise.io/read/01jrymrsgt5y6wtvwkqpx3nvvf))