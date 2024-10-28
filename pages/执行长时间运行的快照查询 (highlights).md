title:: 执行长时间运行的快照查询 (highlights)
author:: [[hbnking]]
full-title:: "执行长时间运行的快照查询"
category:: #articles
url:: https://docs.whaleal.com/mongodb-manual-zh/docs/04-crud/02-query-documents/06-Perform-Long-Running-Snapshot-Queries.html
summary:: 快照查询允许用户从MongoDB读取某个时间点的数据，而不干扰写入工作负载。使用读取关注“snapshot”，可以确保多个相关查询返回一致的数据视图。此功能适用于需要长时间运行且需要数据一致性的查询，避免了不一致的结果。
![](https://readwise-assets.s3.amazonaws.com/static/images/article3.5c705a01b476.png)

- Highlights first synced by [[Readwise]] [[Oct 5th, 2024]]
	- 当 MongoDB 使用默认的读取关注点执行长时间运行的查询时 [`"local"`](https://www.mongodb.com/docs/manual/reference/read-concern-local/#mongodb-readconcern-readconcern.-local-)，查询结果可能包含与查询同时发生的写入数据。因此，查询可能会返回意外或不一致的结果。
	  
	  为了避免这种情况，请创建一个[会话](https://www.mongodb.com/docs/manual/core/read-isolation-consistency-recency/#std-label-sessions)并指定读取关注[`"snapshot"`](https://www.mongodb.com/docs/manual/reference/read-concern-snapshot/#mongodb-readconcern-readconcern.-snapshot-)。通过读取关注 [`"snapshot"`](https://www.mongodb.com/docs/manual/reference/read-concern-snapshot/#mongodb-readconcern-readconcern.-snapshot-)，MongoDB 使用快照隔离来运行您的查询，这意味着您的查询读取最近单个时间点出现的数据。 ([View Highlight](https://read.readwise.io/read/01j8m11gdtr4rpjt4y18hy60eh))
	- 配置快照保留
	  
	  默认情况下，WiredTiger 存储引擎保留历史记录 300 秒。从会话中第一次操作到最后一次操作，您可以使用一个会话`snapshot=true`总共 300 秒。如果您使用会话的时间较长，则会话会失败并出现错误`SnapshotTooOld`。同样，如果使用读关注查询数据，[`"snapshot"`](https://www.mongodb.com/docs/manual/reference/read-concern-snapshot/#mongodb-readconcern-readconcern.-snapshot-)并且查询持续时间超过 300 秒，则查询失败。
	  
	  如果您的查询或会话运行时间超过 300 秒，请考虑增加快照保留期。要延长保留期限，请修改该[`minSnapshotHistoryWindowInSeconds`](https://www.mongodb.com/docs/manual/reference/parameters/#mongodb-parameter-param.minSnapshotHistoryWindowInSeconds) 参数。
	  
	  例如，此命令将 的值设置 [`minSnapshotHistoryWindowInSeconds`](https://www.mongodb.com/docs/manual/reference/parameters/#mongodb-parameter-param.minSnapshotHistoryWindowInSeconds)为 600 秒：
	  
	    db.adminCommand( { setParameter: 1, minSnapshotHistoryWindowInSeconds: 600 } )
	    
	  
	  > 重要的
	  > 
	  > 修改[`minSnapshotHistoryWindowInSeconds`](https://www.mongodb.com/docs/manual/reference/parameters/#mongodb-parameter-param.minSnapshotHistoryWindowInSeconds)为 [MongoDB Atlas](https://www.mongodb.com/docs/atlas/)集群，您必须联系[Atlas支持。](https://www.mongodb.com/docs/atlas/support/) ([View Highlight](https://read.readwise.io/read/01j8m14qqj3eravvm6vtjh83n2))