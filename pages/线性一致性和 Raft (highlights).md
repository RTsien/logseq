title:: 线性一致性和 Raft (highlights)
author:: [[pingcap.com]]
full-title:: "线性一致性和 Raft"
category:: #articles
url:: https://pingcap.com/blog-cn/linearizability-and-raft/
![](https://readwise-assets.s3.amazonaws.com/static/images/article1.be68295a7e40.png)

- Highlights first synced by [[Readwise]] [[Nov 26th, 2023]]
	- 以 TiKV 为例，TiKV 内部可分成多个模块，Raft 模块，RocksDB 模块，两者通过 Log 进行交互，整体架构如下图所示，consensus 就是 Raft 模块，state machine 就是 RocksDB 模块。 ([View Highlight](https://read.readwise.io/read/01hg3rhcgrvwqpscvjcfn3vccb)) #[[tikv]] #[[raft]]
	- Client 将请求发送到 Leader 后，Leader 将请求作为一个 Proposal 通过 Raft 复制到自身以及 Follower 的 Log 中，然后将其 commit。TiKV 将 commit 的 Log 应用到 RocksDB 上，由于 Input（即 Log）都一样，可推出各个 TiKV 的状态机（即 RocksDB）的状态能达成一致。但实际多个 TiKV 不能保证同时将某一个 Log 应用到 RocksDB 上，也就是说各个节点不能**实时**一致，加之 Leader 会在不同节点之间切换，所以 Leader 的状态机也不总有最新的状态。Leader 处理请求时稍有不慎，没有在最新的状态上进行，这会导致整个系统违反线性一致性。**好在有一个很简单的解决方法：依次应用 Log，将应用后的结果返回给 Client。** ([View Highlight](https://read.readwise.io/read/01hg3rhw4sk6wtrcbpb0rjkc6d)) #[[tikv]] #[[raft]]
	- 这样的读简称 LogRead。由于读请求不改变状态机，这个实现就显得有些“重“，不仅有 RPC 开销，还有写 Log 开销。优化的方法大致有两种：
	  
	  •   ReadIndex
	    
	  •   LeaseRead ([View Highlight](https://read.readwise.io/read/01hg3rnaegm73jawtpykqr1ces)) #[[raft]] #[[tikv]]
	- ReadIndex
	  
	  相比于 LogRead，ReadIndex 跳过了 Log，节省了磁盘开销，它能大幅提升读的吞吐，减小延时（但不显著）。Leader 执行 ReadIndex 大致的流程如下：
	  
	  1.  记录当前的 commit index，称为 ReadIndex
	    
	  2.  向 Follower 发起一次心跳，如果大多数节点回复了，那就能确定现在仍然是 Leader
	    
	  3.  等待状态机**至少**应用到 ReadIndex 记录的 Log
	    
	  4.  执行读请求，将结果返回给 Client ([View Highlight](https://read.readwise.io/read/01hg3rq4vjxfmdtnvns0ggdm2z)) #[[raft]] #[[tikv]]
	- LeaseRead
	  
	  LeaseRead 与 ReadIndex 类似，但更进一步，不仅省去了 Log，还省去了网络交互。它可以大幅提升读的吞吐也能显著降低延时。基本的思路是 Leader 取一个比 Election Timeout 小的租期，在租期不会发生选举，确保 Leader 不会变，所以可以跳过 ReadIndex 的第二步，也就降低了延时。 LeaseRead 的正确性和时间挂钩，因此时间的实现至关重要，如果漂移严重，这套机制就会有问题。 ([View Highlight](https://read.readwise.io/read/01hg3rtf3ggtc66zgs0121q0gt)) #[[raft]] #[[tikv]]
	- Wait Free
	  
	  到此为止 Lease 省去了 ReadIndex 的第二步，实际能再进一步，省去第 3 步。这样的 LeaseRead 在收到请求后会立刻进行读请求，不取 commit index 也不等状态机。由于 Raft 的强 Leader 特性，在租期内的 Client 收到的 Resp 由 Leader 的状态机产生，所以只要状态机满足线性一致，那么在 Lease 内，不管何时发生读都能满足线性一致性。有一点需要注意，只有在 Leader 的状态机应用了当前 term 的第一个 Log 后才能进行 LeaseRead。因为新选举产生的 Leader，它虽然有全部 committed Log，但它的状态机可能落后于之前的 Leader，状态机应用到当前 term 的 Log 就保证了新 Leader 的状态机一定新于旧 Leader，之后肯定不会出现 stale read。 ([View Highlight](https://read.readwise.io/read/01hg3rv5q8srw9y6t5atk57jj3)) #[[raft]] #[[tikv]]
	- 以只要状态机满足线性一致，那么在 Lease 内，不管何时发生读都能 ([View Highlight](https://read.readwise.io/read/01hg3rx3bz4nrgyj8rkac048fg))