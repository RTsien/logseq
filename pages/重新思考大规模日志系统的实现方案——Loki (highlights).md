title:: 重新思考大规模日志系统的实现方案——Loki (highlights)
author:: [[woa.com]]
full-title:: "重新思考大规模日志系统的实现方案——Loki"
category:: #articles
url:: https://km.woa.com/articles/show/546946?kmref=author_recommend
![](https://readwise-assets.s3.amazonaws.com/static/images/article3.5c705a01b476.png)

- Highlights first synced by [[Readwise]] [[Nov 28th, 2023]]
	- 日志系统除了**低频查询**和**不可变数据**以外，其实还有个显著的特点，就是几乎所有查询都是时间相关的 ([View Highlight](https://read.readwise.io/read/01hg9160qjxzxps4smrh6apzbj))
		- **Note**: 与搜索引擎的区别
	- 从功能性上来讲，grep其实表达能力更强，没有啥是你不能搜的。而ES和索引有关系，而索引又和分词强相关。如果分词没有切割出某个term，你想搜就比较麻烦了，甚至没法搜。比如对于日志 `hello-world` ，你想搜 `o-w` ，grep一点问题都没有，而ES则… ([View Highlight](https://read.readwise.io/read/01hg91cn7zqjbtjar0d02ctm90))
	- Ingester是WritePath中最核心的模块，主要负责把日志做batch并持久化到外部存储，比如S3(cos) hbase 或者 cassandra 等等，它主要做以下事情：
	  
	  •   维护自己在hash环中的状态，新节点要自动加入，问题节点要平滑退出等等，方便Distributor找到自己
	  •   内存中维护多个log stream，把新的日志写入对应的stream，并定期以chunk为单位持久化到外部存储
	  •   WAL，避免内存中日志flush到外部存储前由于崩溃而丢失 ([View Highlight](https://read.readwise.io/read/01hg91v8x9v75wnm4vv45z3h98))
- New highlights added [[Nov 28th, 2023]] at 10:16 AM
	- 一个大规模日志系统更贴近实际的技术需求指标，其中的核心就是Immutable和低频查询。顺着这个思路，我们引出了Loki，而它的解决办法就是 部分索引 + 并发grep。Loki把日志压缩存储到对象存储COS中，能够极大的节约成本，而并发grep在低频查询的场景下也是完全可行的，并且计算节点可以无限扩容。而Loki的索引使用boltdb存储，boltdb又定期存储到COS中，这样Loki整体可以只依赖一个COS，部署起来依赖很少。由于日志Immutable的特性，Loki可以大量使用缓存来提高查询效率，这就几乎可以抹平和ES的效率差距 ([View Highlight](https://read.readwise.io/read/01hg9tf062dkbjaj2g22vca59y))
	- 当了解了Loki之后，我发现其实Trace数据也是低频查询+不可变的特征，似乎也可以这么改进。进而发现grafana真的还有个新的系统——[Tempo](https://km.woa.com/articles/show/546946/%5BGrafana%20Tempo%20%7C%20Grafana%20Labs%5D%28https://grafana.com/oss/tempo/))，使用和Loki几乎一样的架构，也是把Trace存储到对象存储中，从而极大地降低Trace的成本。tempo的目标就是要低成本地支持100% Trace上报，而不是采样。这样，Trace就可以成为我们问题排查的利器，而不仅仅是通过采样数据查看性能瓶颈。 ([View Highlight](https://read.readwise.io/read/01hg9tg3kry813jeq7vz7k81j2))