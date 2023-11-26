title:: MySQL Online DDL 探究 (highlights)
author:: [[Everyday]]
full-title:: "MySQL Online DDL 探究"
category:: #articles
url:: https://huangzhw.github.io/2018/09/20/mysql-online-ddl/
![](https://readwise-assets.s3.amazonaws.com/static/images/article2.74d541386bbf.png)

- Highlights first synced by [[Readwise]] [[Nov 26th, 2023]]
	- 在复制架构下的官方Online DDL的问题
	  
	  在通读了MySQL Online DDL的文档后，我觉得这个工具非常适合我们，就使用他来加索引。
	  
	  但是有一次对一个千万级别的大表加索引时，出现了用户购买成功但是没有获取到VIP的情况，但是购买成功了VIP是肯定加上了的。因为我们的架构读数据是在`Slave`读取的，所以只能是主从同步出问题了。查看了监控图标，发现`Slave`在一段时间内有大量的复制延迟。
	  
	  在网上查阅了资料，发现原因如下：
	  
	  1.  在`Master`上执行DDL语句时，这时候允许并行的DML操作没有什么问题；
	  2.  但是在`Master`的DDL语句没有执行完前，这条语句是不会同步到`Slave`的，执行完后，这条语句同步到`Slave`开始执行；
	  3.  在`Slave`上执行DDL时，在DDL之后的DML语句不会被执行，直到DDL执行完毕后，这些DML语句才会开始执行；
	  4.  然后`Slave`需要一段时间跟上`Master`。
	  
	  `Slave`不能并行执行的原因是这些DML操作语句可能依赖于表的Schema的修改。 ([View Highlight](https://read.readwise.io/read/01hg3m8886zy4erb4tqhep6hdc))
	- 当`ALTER`语句完成的瞬间： ([View Highlight](https://read.readwise.io/read/01hg3md83hbcwg473y2g6a2nbs))
	- 以上说明，在`Master`执行完成，将语句同步到`Slave`后，`Slave`后续的DML都卡主了。 ([View Highlight](https://read.readwise.io/read/01hg3mav8jq245hgf2q6qy8yx6))
	- 在`Master-Slave`架构下，官方的Online DDL存在致命的缺陷，所以我们只能转向第三方工具。这里有Percona的公司的pt-osc，是基于触发器的Online DDL的代表。以及比较新的，Github的gh-ost，是基于Binlog的。 ([View Highlight](https://read.readwise.io/read/01hg3mg8zw18e2sr3xdf78afkr))