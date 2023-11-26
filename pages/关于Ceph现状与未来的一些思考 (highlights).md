title:: 关于Ceph现状与未来的一些思考 (highlights)
author:: [[袁冬]]
full-title:: "关于Ceph现状与未来的一些思考"
category:: #articles
url:: http://mp.weixin.qq.com/s?__biz=MjM5MDE0Mjc4MA==&mid=206915323&idx=1&sn=eb4663de50f1d2780e5839813631de06#rd
tags:: #[[favorite]] 
![](http://mmbiz.qpic.cn/mmbiz/YriaiaJPb26VOmJIBgff4hrGPibIpt37ibzy09nOMuqEPibu9myns4pgubWjHqjhRG2YbmSn6aTcCrQX2LVvbicKYKpw/0?wx_fmt=jpeg)

- Highlights first synced by [[Readwise]] [[Nov 24th, 2023]]
	- CRUSH算法是Ceph最初的两大创新之一（另一个是基于动态子树分区的元数据集群），也是整个RADOS的基石，是Ceph最引以为豪的地方。 ([View Highlight](https://read.readwise.io/read/01hg07cfa9vmrmj999fqsszacb))
	- Ceph的特性不可谓不多，从分布式系统最基本的横向扩展、动态伸缩、冗余容灾、负载平衡等，到生产环境环境中非常实用的滚动升级、多存储池、延迟删除等，再到高大上的CephFS集群、快照、纠删码、跨存储池缓存等，不可谓不强大。 ([View Highlight](https://read.readwise.io/read/01hg07ewkzmfdyyyvk968cdfhh))
	- Ceph中的RADOS采用强一致性设计，即Write-All-Read-One，这种模式的好处在于读取效率较高，而且工程难度较低，比较适合与读多写少的系统。 ([View Highlight](https://read.readwise.io/read/01hg07wnem2wv3ctvfgd2aad7s))
- New highlights added [[Nov 24th, 2023]] at 7:25 PM
	- Write-All-Read-One对于硬件可靠性的要求几乎是无法满足的。想象一下一个10PB的系统，按照最大4TB每块盘的计算，就有2500块磁盘。按照我们以往的运维经验，每周存在一块磁盘故障是完全正常的。这种场景下，如果数据分布足够分散，实际上一块磁盘可能涉及到很多数据块，也就是说一块磁盘故障会影响很多IO，而这种情况每周发生一次。这对业务连续性的影响是已经是不可忽略的。 ([View Highlight](https://read.readwise.io/read/01hg0gn6611hav1s10pad28jmv))