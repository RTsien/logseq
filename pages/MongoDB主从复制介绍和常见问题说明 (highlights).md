title:: MongoDB主从复制介绍和常见问题说明 (highlights)
author:: [[O]]
full-title:: "MongoDB主从复制介绍和常见问题说明"
category:: #articles
url:: https://cloud.tencent.com/developer/article/1592500
summary:: The text explains common misunderstandings and issues related to MongoDB replica set primary-secondary replication, focusing on topics like data syncing and delays between nodes. It also discusses how to optimize replication processes, such as adjusting thread counts and batch limits, to improve performance and avoid potential bottlenecks. Additionally, it highlights the importance of understanding and managing replication delays, as well as the impact of chaining replication on system performance and data consistency in MongoDB setups.
![](https://readwise-assets.s3.amazonaws.com/media/uploaded_book_covers/profile_1160846/cloud_vcWA62g.png)

- Highlights first synced by [[Readwise]] [[May 30th, 2024]]
	- [MongoDB副本集](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fdocs.mongodb.com%2Fmanual%2Freplication%2F&source=article&objectId=1592500)模式下，用户向主节点写入数据，并记录oplog. 从节点通过oplog进行数据同步，最终保证副本集中的各个节点的数据一致性。
	  
	  客户端可以指定写入请求的一致性级别（WriteConcern），比如对于数据一致性较高的场景，可以设置数据复制到“大多数”节点才返回成功。这样能够保证即使主节点重启后不会回滚掉之前写入的数据。
	  
	  一个常见的误解：
	  
	  “写大多数”请求的流程如下，客户端只需要向主节点写入数据即可（不需也不能向从节点直接写数据）；从节点进行oplog同步之后，会将自身已经同步的oplog时间点通知给主节点；主节点维护了副本集中各个从节点的oplog同步情况，如果确定数据已经到了大多数节点上（包括自己），则给客户端返回成功。如果数据同步发生了异常，或者同步太慢，则可能触发超时。
	  
	  > 同理，[ReadConcern Majority](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fdocs.mongodb.com%2Fmanual%2Freference%2Fread-concern%2F&source=article&objectId=1592500)也不是客户端去读多个节点，这里不详细讨论 ([View Highlight](https://read.readwise.io/read/01hz03vxfgpv7hbmadf65fss0y))
	- ![副本集数据同步示意图](https://ask.qcloudimg.com/draft/2518717/z3wkk5yuuc.png) ([View Highlight](https://read.readwise.io/read/01hz03w32388a1av1w814zenkp))
	- ![主从复制细节](https://ask.qcloudimg.com/draft/2518717/20jq7bq4ya.png) ([View Highlight](https://read.readwise.io/read/01hz03weqhf55v0fdpjkdhzw2x))