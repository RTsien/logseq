title:: 如何在不停机情况下，给MySQL一张千万数据量表加索引？ (highlights)
author:: [[woa.com]]
full-title:: "如何在不停机情况下，给MySQL一张千万数据量表加索引？"
category:: #articles
url:: https://mk.woa.com/q/292363?ADTAG=daily
![](https://readwise-assets.s3.amazonaws.com/static/images/article0.00998d930354.png)

- Highlights first synced by [[Readwise]] [[Dec 15th, 2023]]
	- 如果大表无主键索引会影响主备同步效率，后期线上ddl时可以在闲时让dba先在备库上加，操作之前先关闭binlog，加完之后主库无大事物时切换主备数据源，然后再在主库同样方式加索引。
	  
	  评论 2 收起 12月13日 10:33
	  
	  [![](https://mk.woa.com/avatar/user/gengkaiyu)](https://mk.woa.com/u/gengkaiyu)
	  
	  [gengkaiyu · PCG](https://mk.woa.com/u/gengkaiyu)
	  
	  请问这里为什么要关闭binlog？
	  
	  回复 12月14日 17:42
	  
	  [![](https://mk.woa.com/avatar/user/chrispfwu)](https://mk.woa.com/u/chrispfwu)
	  
	  [chrispfwu · CSIG](https://mk.woa.com/u/chrispfwu)
	  
	  [@gengkaiyu](https://mk.woa.com/u/gengkaiyu) 主库有业务在用呢，在备库上加索引关闭binlog不会同步ddl操作到主库造成主库锁表。简单就是一句话主备库交叉在slave上执行ddl，这样不影响业务。所有操作的前提是主备数据同步无延迟，无大事物。 ([View Highlight](https://read.readwise.io/read/01hhp5y6j8m8t181e2n30w5h1v))
	- 贴一下过去的经验：假设你的是MySQL5.7或以上，mysql都是支持online ddl的，也就是可以不阻塞正常的dml读写语句，不影响业务。不过有几个前提
	  
	  1.  ddl启动和结束的时候都要获取exclusive metadata lock
	  2.  获取metadata lock时不能有dml在跑
	  3.  所以只要确保没有长事务就可以  
	    可以用SQL确认有无长事务，开始alter变更后也可以看是否在等待mdl，如果在等mdl过久的话需要停止ddl。**等mdl的过程会阻塞dml！**
	  
	    select * from information_schema.innodb_trx where trx_started<now()-interval 1 second;
	    select * from information_schema.processlist where lower(STATE) like '%wait%'; ([View Highlight](https://read.readwise.io/read/01hhp5zxnz5zpca0zbgvkffbvt))