title:: 【Agones系列】Agones总结与展望 (highlights)
author:: [[知乎专栏]]
full-title:: "【Agones系列】Agones总结与展望"
category:: #articles
url:: https://zhuanlan.zhihu.com/p/537533305
![](https://picx.zhimg.com/v2-d399b2e0e289f3ccdec0564f41681de0_720w.jpg?source=172ae18b)

- Highlights first synced by [[Readwise]] [[Jan 12th, 2024]]
	- Agones是针对Delicated Game Server而设计的云原生负载，比较适合MOBA/FPS/TCG等有对局概念、需要匹配的游戏类型。这类游戏的特点就是生命周期短，更偏向无状态，可以快速地构建、删除Pod，实现资源的最大程度利用。 ([View Highlight](https://read.readwise.io/read/01hkwd017mwhvefaygcgkwnm53))
	- Agones的核心是提供了动态伸缩的能力，然而想要最大程度利用Agones伸缩性，需要配合OpenMatch使用。
	  
	  OpenMatch是Google开源的另一项目，它提供了一种匹配机制，用户可自定义匹配方法在这套框架下实现对局的匹配。从实现逻辑上来看，Agones这部分是比较轻量的，重头部分在于OpenMatch提供的匹配能力，OpenMatch相对与Agones显得非常厚重，用户要想接入需要进行相对复杂的工作，也有不少公司在这部分做文章提供Saas服务，简化客户端接入。 ([View Highlight](https://read.readwise.io/read/01hkwcwzvpa7qsgs7jfv5c0tdq))