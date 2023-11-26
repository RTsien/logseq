title:: TiKV 源码解析系列 - 如何使用 Raft (highlights)
author:: [[xuliwen]]
full-title:: "TiKV 源码解析系列 - 如何使用 Raft"
category:: #articles
url:: https://pingcap.com/blog-cn/tikv-how-to-use-raft/
![](https://img1.www.pingcap.com/prod/1_52231c6472.png)

- Highlights first synced by [[Readwise]] [[Nov 26th, 2023]]
	- TiKV 以及 etcd 对于 membership change 的处理，跟 Raft 论文是稍微有一点不一样的，主要在于 TiKV 的 membership change 只有在 log applied 的时候生效，这样主要的目的是为了实现简单，但有一个风险在于如果我们只有两个节点，要从里面移掉一个节点，如果一个 follower 还没收到 ConfChange 的 log entry，leader 就当掉并且不可恢复了，整个集群就没法工作了。所以通常我们都建议用户部署 3 个或者更多个奇数个节点。 ([View Highlight](https://read.readwise.io/read/01hg3qvchr0rh6stg22dwtem1c)) #[[raft]] #[[tikv]]