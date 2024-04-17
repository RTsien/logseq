title:: 十倍性价比蜕变：腾讯云es全新架构助力日志场景降本增效 (highlights)
author:: [[woa.com]]
full-title:: "十倍性价比蜕变：腾讯云es全新架构助力日志场景降本增效"
category:: #articles
url:: https://km.woa.com/articles/show/600762?jumpFrom=BBS_MAIL&jumpfrom=systemmail
summary:: The text discusses how Tencent Cloud's ES architecture, leveraging technologies like storage-compute separation and parallel querying, helps improve efficiency and reduce costs in log scenarios. By implementing strategies like cold-hot data separation and parallel IO operations, Tencent Cloud ES achieves significant performance gains and cost savings compared to traditional architectures. The text also hints at future optimizations, such as aggregation pushdown and sorting enhancements, to further enhance performance and cost-effectiveness in log scenarios.
![](https://readwise-assets.s3.amazonaws.com/static/images/article3.5c705a01b476.png)

- Highlights first synced by [[Readwise]] [[Apr 12th, 2024]]
	- 3.1、存算分离3.1.1、设计思想
	  
	       ![](https://km.woa.com/asset/00010002240400ea179b3725e8436301?height=862&width=1436)
	  
	  混合存储引擎的整体设计思想是基于典型的 delta + base 架构。其中 delta 部分我们采用 SSD，主要目的为了扛高并发写入，以及小 segment 的存储及合并；而 base 部分采用对象存储，用于存储大量不可变的大 segment，一方面其高可用（4个9一个5）、高可靠（12个9）、按量付费、免运维的架构降低了大量运维成本，更重要的是其提供的标准、低频、归档等灵活的低成本存储方案能大幅降低海量日志的存储成本。 ([View Highlight](https://read.readwise.io/read/01hv6seg94hr6awcsczeda3dk1))
	- ![](https://km.woa.com/asset/000100022404007de9cfc052174db601?height=748&width=1566)
	  
	  第六个阶段，本地 segment 通过 merge 产生了较大的 segment，会被冻结不再参与 merge，并下沉至底层共享存储。注意这个阶段是从热数据就开始的，并不是数据降温后才启动下沉，可以避免数据降温后整体下沉的排队拥塞，当然可以根据用户的需求进行灵活配置。此时，日志场景大量的查询会集中在本地，本地 primary 和 replica 也能很好的扛住读写压力。
	  
	  第七个阶段，数据的查询频次有所降低。一般情况，本地的 primary 即可满足绝大部分查询性能需求。此时 replica 会从本地卸载，读取会走远端共享存储，同时本地会有缓存机制保存用户常用查询数据提升性能。此时的查询会优先打到 primary。**通过卸载本地 replica，我们可以缩减约 50% 的 SSD 容量。** ([View Highlight](https://read.readwise.io/read/01hv6smyw6avzj1x6tmj98hczs))
		- 💡: 共享读
	- segment形态
	  
	  在整个数据生命周期中，segment 呈现三种形态：
	  
	  1、  Local Segment：行列存、索引文件等全部在本地，抗住热数据高并发读写请求。
	  
	  2、  Mixed Segment：行列存等数据文件可能卸载，只有部分索引文件在本地，满足少量的查询请求。
	  
	  3、  Remote Segment：行列存、索引文件等全部在远程，本地只有少量元数据文件，满足冷数据低成本归档需求。 ([View Highlight](https://read.readwise.io/read/01hv6spr7n2reqxmtn0n9g9z93))
	- ES写入数据时最终是通过Lucene写入到内存中，一段时间后refresh成segment，我们可以在外部 (flink、spark、独立ES集群等) 提前通过Lucene的api创建好segment发送给ES，ES接收到内存segment后转发到索引分片对应的数据节点中，索引分片直接追加到lucene中即可，因为ES的refresh(Lucened的flush)也是类似的原理。 ([View Highlight](https://read.readwise.io/read/01hv6sr4rc0d17mt08rwj553q8))
		- 💡: 需要进一步阅读一下本章内容 #TODO