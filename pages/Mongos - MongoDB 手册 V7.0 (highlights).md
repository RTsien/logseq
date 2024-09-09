title:: Mongos - MongoDB 手册 V7.0 (highlights)
author:: [[mongodb.com]]
full-title:: "Mongos - MongoDB 手册 V7.0"
category:: #articles
url:: https://www.mongodb.com/zh-cn/docs/manual/core/sharded-cluster-query-router/#std-label-mongos-hedged-reads
![](https://readwise-assets.s3.amazonaws.com/media/uploaded_book_covers/profile_1160846/meta_generic.png)

- Highlights first synced by [[Readwise]] [[Sep 10th, 2024]]
	- 对冲读 (Hedged Reads)[](https://www.mongodb.com/zh-cn/docs/manual/core/sharded-cluster-query-router#hedged-reads)
	  
	  [`mongos`](https://www.mongodb.com/zh-cn/docs/manual/reference/program/mongos#mongodb-binary-bin.mongos) 实例可以对冲使用非 `primary`[读取偏好](https://www.mongodb.com/zh-cn/docs/manual/core/read-preference/)的读取。通过对冲读，[`mongos`](https://www.mongodb.com/zh-cn/docs/manual/reference/program/mongos#mongodb-binary-bin.mongos) 实例将读取操作路由到每个查询分片的两个副本集成员，并从每个分片的第一个响应者返回结果。为对冲读取操作而发送的其他读取使用 [`maxTimeMSForHedgedReads`](https://www.mongodb.com/zh-cn/docs/manual/reference/parameters#mongodb-parameter-param.maxTimeMSForHedgedReads) 的 `maxTimeMS`[](https://www.mongodb.com/zh-cn/docs/manual/reference/parameters#mongodb-parameter-param.maxTimeMSForHedgedReads) 值。 ([View Highlight](https://read.readwise.io/read/01j7711k87x6nq012gdqmb4vay))