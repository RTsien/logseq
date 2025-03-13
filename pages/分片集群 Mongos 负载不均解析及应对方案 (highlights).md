title:: 分片集群 Mongos 负载不均解析及应对方案 (highlights)
author:: [[tencent.com]]
full-title:: "分片集群 Mongos 负载不均解析及应对方案"
category:: #articles
url:: https://cloud.tencent.com/document/product/240/81195
summary:: The document discusses load balancing issues in MongoDB sharded clusters, particularly related to Mongos nodes handling queries. It explains that to ensure proper functioning of cursor-based operations and transactions, requests must be routed to the same Mongos node. Users can customize their connection settings to manage load distribution effectively, especially in scenarios with fewer source IPs.
![](https://cloudcache.tencent-cloud.com/open_proj/proj_qcloud_v2/gateway/shareicons/cloud.png)

- Highlights first synced by [[Readwise]] [[Nov 20th, 2024]]
	- 批量扫描 getMore 问题
	  
	  当 MongoDB 无法将 find 结果一次性返回时，会优先返回第一批数据 + cursorID，客户端通过这个 cursorID 不断 getMore 迭代剩余的数据。所以一次批量扫描请求可能会对应1次 find 请求和多次 getMore 请求，并通过 cursorID 关联。 ([View Highlight](https://read.readwise.io/read/01jd1f9d193jxzh4h3dd248tn5))
	- 事务操作问题
	  
	  MongoDB 在 4.2 版本支持了分布式事务，用户可以连接 Mongos 节点发起事务操作。在 startTransaction 和 commitTransaction/abortTransaction 之间可以执行多次 [读写操作](https://www.mongodb.com/docs/manual/core/transactions-operations/)，Mongos 在内存中记录了事务中每次请求携带的 logicalSessionId 和 txnId 等元数据来维护上下文关系。因此，[MongoDB 的设计](https://github.com/mongodb/mongo/blob/master/src/mongo/db/s/README.md#transactions) 决定了需要保证事务中的每个操作都发到同一个 Mongos 上执行。 ([View Highlight](https://read.readwise.io/read/01jd1f92wtaka8m9cxqwqt59z9))
	- 云数据库 MongoDB 负载均衡策略
	  
	  基于批量扫描 getMore 问题及其事务操作问题的考虑，云数据库 MongoDB 负载均衡 Hash 策略根据访问端（一般是 CVM）IP 信息来均衡分流：一个源 IP 的请求都会落在同一个 Mongos 上，保证 getMore() 和事务操作在同一个上下文进行。
	  
	  一般生产环境中访问端 IP 较多，这种策略效果较好。但是当访问端 IP 较少的情况下，特别是在压测场景下，容易引起 Mongos 负载不均衡的问题。 ([View Highlight](https://read.readwise.io/read/01jd1fad1t4vep9wfnzrtx0yq9))