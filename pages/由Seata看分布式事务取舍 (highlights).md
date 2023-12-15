title:: 由Seata看分布式事务取舍 (highlights)
author:: [[简书]]
full-title:: "由Seata看分布式事务取舍"
category:: #articles
url:: https://www.jianshu.com/p/917cb4bdaa03
![](https://upload-images.jianshu.io/upload_images/3751238-dbc539a8f64b233f.png)

- Highlights first synced by [[Readwise]] [[Dec 14th, 2023]]
	- 基于数据库的XA协议本质上就是两阶段提交，但由于性能原因在互联网高并发场景下并不适用。如果数据库只能保证本地ACID时，那么出现其中交易异常后，如何实现整个交易原子性A，从而保证一致性C呢？另外在处理过程中如何保证隔离性呢？  
	  最直接就是按照逻辑依次调用服务，当出现异常怎么办?那就对那些已经成功的进行补偿，补偿成功就一致了，这种朴素的模型就是Saga。但Saga这种方式并不能保证隔离性，于是出现了TCC，在实际交易逻辑前先做业务检查、对涉及到的业务资源进行“预留”，或者说是一种“中间状态”，如果都预留成功则完成这些预留资源的真正业务处理，典型的如票务座位等场景。当然还有像Ebay提出的基于消息表，即可靠消息最终一致模型，但本质上这也属于Saga模式的一种特定实现，它的关键点有两个：1.基于应用共享事务记录执行轨迹，2.然后通过异步重试确保交易最终一致（这也使得这种方式不适用那些业务上允许补偿回滚的场景）。 ([View Highlight](https://read.readwise.io/read/01hhe4afkjwwy4cgrez9jhccqf))
		- 💡: Saga和TCC都会对失败进行重试或者补偿，但是TCC会对资源进行检查和预锁定，相比Saga提高了隔离性。而可靠消息就是靠重试，没有补偿/回退逻辑。
	- 仔细对比这些方案与XA，会发现这些方案本质上都是将两阶段提交从资源层提升到了应用层。  
	  • Saga的核心就是补偿，一阶段就是服务的正常顺序调用（数据库事务正常提交），如果都执行成功，则第二阶段则什么都不做；但如果其中有执行发生异常，则依次调用其补偿服务（一般多逆序调用未已执行服务的反交易）来保证整个交易的一致性。*应用实施成本一般*。  
	  • TCC的特点在于业务资源检查与加锁，一阶段进行校验，资源锁定，如果第一阶段都成功，二阶段对锁定资源进行交易逻辑，否则，对锁定资源进行释放。*应用实施成本较高*。  
	  • 基于可靠消息最终一致，一阶段服务正常调用，同时同事务记录消息表，二阶段则进行消息的投递，消费。*应用实施成本较低* ([View Highlight](https://read.readwise.io/read/01hhe4wy42qgad6xt3z6qj4wb2))
	- 我们看看Seata增加了哪些开销（纯内存运算类的忽略不计）：  
	  一条Update的SQL，则需要全局事务xid获取（与TC通讯）、before image（解析SQL，查询一次数据库）、after image（查询一次数据库）、insert undo log（写一次数据库）、before commit（与TC通讯，判断锁冲突），这些操作都需要一次远程通讯RPC，而且是同步的。另外undo log写入时blob字段的插入性能也是不高的。**每条写SQL都会增加这么多开销,粗略估计会增加5倍响应时间**（二阶段虽然是异步的，但其实也会占用系统资源，网络、线程、数据库）。
	  
	  > 前后镜像如何生成？  
	  > 通过druid解析SQL，然后复用业务SQL中的where条件，然后生成Select SQL执行。 ([View Highlight](https://read.readwise.io/read/01hhe54jz7925jretpbqnd6q7c))
	- Seata的引入全局锁会额外增加死锁的风险，具体可见[https://github.com/seata/awesome-seata/blob/master/wiki/en-us/Fescar-AT.md](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fseata%2Fawesome-seata%2Fblob%2Fmaster%2Fwiki%2Fen-us%2FFescar-AT.md)，但如果出现死锁，会不断进行重试，最后靠等待全局锁超时，这种方式并不优雅，也延长了对数据库锁的占有时间。 ([View Highlight](https://read.readwise.io/read/01hhe55y08am4780txwrysjzvn))
	- 基于XA的分布式事务如果要严格保证ACID，实际需要事务隔离级别为SERLALIZABLE。 ([View Highlight](https://read.readwise.io/read/01hhe56k7z0nksd4bpkjnxgrkt))
	- 一个好的分布式事务框架应用尽可能满足以下特性：  
	  **1. 业务改造成本低**  
	  **2. 性能损耗低**  
	  **3. 隔离性保证完整**  
	  但如同CAP，这三个特性是相互制衡的，往往只能满足其中两个，我们可以画一个三角约束：
	  
	  基于业务补偿的Saga满足1.2；TCC满足2.3；Seata满足1.3。 ([View Highlight](https://read.readwise.io/read/01hhe5745vmz9skx4y05gnpk0a))
	- 无论是数据库领域XA、Google percolator或Calvin模型，还是微服务下Saga、TCC、可靠消息等方案，都没有完美解决分布式事务问题，它们不过是各自在性能、一致性、可用性等方面做取舍，寻求某些场景偏好下的权衡。
	  
	  > **其实由于网络的不确定性，分布式下很多问题都是难题，最好的方案是避免分布式事务:)** ([View Highlight](https://read.readwise.io/read/01hhe58yzsk3t2spb59qndydsa))