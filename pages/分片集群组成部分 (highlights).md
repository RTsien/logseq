title:: 分片集群组成部分 (highlights)
author:: [[mongodb.com]]
full-title:: "分片集群组成部分"
category:: #articles
url:: https://www.mongodb.com/zh-cn/docs/manual/core/sharded-cluster-components/
summary:: MongoDB分片集群由分片、mongos路由器和配置服务器组成。生产环境中建议使用冗余和高可用性配置，通常包括三个节点的副本集。开发环境则可以简化为一个mongos实例和一个分片副本集。
![](https://readwise-assets.s3.amazonaws.com/media/uploaded_book_covers/profile_1160846/meta_generic_EOx3he5.png)

- Highlights first synced by [[Readwise]] [[Sep 10th, 2024]]
	- MongoDB [分片集群](https://www.mongodb.com/zh-cn/docs/manual/reference/glossary#std-term-sharded-cluster)由以下组件构成：
	  
	  •   [分片](https://www.mongodb.com/zh-cn/docs/manual/core/sharded-cluster-shards#std-label-shards-concepts)：每个分片都包含分片数据的一个子集。每个分片都必须作为一个[副本集](https://www.mongodb.com/zh-cn/docs/manual/reference/glossary#std-term-replica-set)进行部署。
	    
	  •   [mongos](https://www.mongodb.com/zh-cn/docs/manual/core/sharded-cluster-query-router/)：`mongos` 充当查询路由器，在客户端应用程序和分片集群之间提供接口。[`mongos`](https://www.mongodb.com/zh-cn/docs/manual/reference/program/mongos#mongodb-binary-bin.mongos) 可以支持[对冲读](https://www.mongodb.com/zh-cn/docs/manual/core/sharded-cluster-query-router#std-label-mongos-hedged-reads)，从而最大限度地降低延迟。
	    
	  •   [配置服务器](https://www.mongodb.com/zh-cn/docs/manual/core/sharded-cluster-config-servers#std-label-sharding-config-server)：配置服务器存储集群的元数据和配置设置。配置服务器必须部署为副本集 (CSRS)。 ([View Highlight](https://read.readwise.io/read/01j771572bxnyr5vf8yy7xscxk))
	- 下图展示了生产中使用的常见分片集群架构：
	  
	  ![](data:image/svg+xml;charset=utf-8,%3Csvg%20height='540'%20width='960'%20xmlns='http://www.w3.org/2000/svg'%20version='1.1'%3E%3C/svg%3E)
	  
	  ![Diagram that shows a production-level sharded cluster
	  containing multiple shards and mongos routers.](https://www.mongodb.com/zh-cn/docs/manual/static/1112d075b61fb59a49076c865c6e8f60/bde8a/sharded-cluster-production-architecture.webp)
	  
	  ![Diagram that shows a production-level sharded cluster
	  containing multiple shards and mongos routers.](https://www.mongodb.com/zh-cn/docs/manual/static/1112d075b61fb59a49076c865c6e8f60/bde8a/sharded-cluster-production-architecture.webp) ([View Highlight](https://read.readwise.io/read/01j771695nwqs6vm1h5efm6j21))