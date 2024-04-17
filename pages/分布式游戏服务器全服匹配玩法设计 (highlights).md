title:: 分布式游戏服务器全服匹配玩法设计 (highlights)
author:: [[woa.com]]
full-title:: "分布式游戏服务器全服匹配玩法设计"
category:: #articles
url:: https://km.woa.com/articles/show/411161?kmref=search&from_page=1&no=10
summary:: The text discusses the design of a matchmaking system for distributing game servers, focusing on offline matching for large amounts of data. It covers the challenges of matching numerous groups efficiently and suggests solutions such as storing key information in a database and implementing a batching matching process. The text also explores the importance of maintaining match nodes, efficient matching strategies, system redundancy, and implementing matching rules effectively.
![](https://readwise-assets.s3.amazonaws.com/static/images/article1.be68295a7e40.png)

- Highlights first synced by [[Readwise]] [[Apr 16th, 2024]]
	- 那么如何动态选择某个团体服务器进行匹配呢？ 这里我们可以采用**多个进程****抢占乐观锁的方式争夺匹配权**，保证同一时间只有其中一个进程进行全服匹配；同时需要增加超时处理机制，当某个进程正在匹配中，发生了宕机导致匹配超时，会有其他进程继续抢占乐观锁将匹配流程进行下去。进程抢占匹配权流程如下：
	  
	  ![](https://km.woa.com/asset/68891429fb7340dcabfa59ade61e843b?height=725&width=430)
	  
	  由于匹配进程存在中途宕机的可能，分批匹配的过程中我们需要在<分组,随机>步骤和<匹配>步骤完成后及时将分组顺序，匹配进度，上批匹配完成时间等信息存入DB，保证异常情况匹配流程能够顺利进行下去。 ([View Highlight](https://read.readwise.io/read/01hvgrwjpzavjejn249n7bd6yr))
	- 如果说遍历删除属于一种短视的贪心算法，那么匈牙利算法则是一种更有远见的可以补救短视的贪心算法。
	  
	  在介绍匈牙利算法之前，首先介绍两个重要的名词：
	  
	  •   交错路 ：从一个未匹配点出发，依次遍历未匹配边、匹配边、未匹配边，这样交替下去，这条路径称为交错路。
	  •   增广路 ：从一个未匹配点出发，依次遍历未匹配边、匹配边、未匹配边，这样交替下去，如果最后一个点是未匹配点，这条路径称为增广路。换句话说，起点和终点都为未匹配点的交错路为增广路。
	  
	  匈牙利算法的核心思想就是**为图中每个未匹配节点寻找一次增广路，然后再对已有匹配关系进行替换**。我们仍以上一张图举例： ([View Highlight](https://read.readwise.io/read/01hvgs1drze8zn3tvhepaj97s7))