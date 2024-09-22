title:: MongoDB特定场景性能数十倍提升优化实践 (highlights)
author:: [[mongoingcom@126.com]]
full-title:: "MongoDB特定场景性能数十倍提升优化实践"
category:: #articles
url:: https://mongoing.com/archives/75162
summary:: A core Java service using MongoDB faced severe performance issues, including a "snowball" failure that caused traffic to drop to zero. The problems stemmed from improper client configurations and issues with MongoDB's handling of connections, leading to excessive load on the proxies. After extensive analysis, solutions included optimizing client settings and improving the MongoDB kernel to handle random number generation more efficiently.
![](https://readwise-assets.s3.amazonaws.com/static/images/article1.be68295a7e40.png)

- Highlights first synced by [[Readwise]] [[Sep 10th, 2024]]
	- 分析当时的代理QPS监控，正常query读请求的QPS访问曲线如下，故障时间段QPS几乎跌零雪崩了：
	  
	  [![](https://mongoing.com/wp-content/uploads/2020/10/1f0e3dad9990834.jpg)](https://mongoing.com/wp-content/uploads/2020/10/1f0e3dad9990834.jpg)
	  
	  Command统计监控曲线如下：
	  
	  [![](https://mongoing.com/wp-content/uploads/2020/10/98f13708210194c.jpg)](https://mongoing.com/wp-content/uploads/2020/10/98f13708210194c.jpg)
	  
	  从上面的统计可以看出，当该代理节点的流量故障时间点有一波尖刺，同时该时间点的command统计瞬间飙涨到22000(实际可能更高，因为我们监控采样周期30s,这里只是平均值)，也就是瞬间有2.2万个连接瞬间进来了。Command统计实际上是db.ismaster()统计，客户端connect服务端成功后的第一个报文就是ismaster报文，服务端执行db.ismaster()后应答客户端，客户端收到后开始正式的sasl认证流程。 ([View Highlight](https://read.readwise.io/read/01j773ng4ncqs1ves4qxc9mm8n))
	- 正常客户端访问流程如下：
	  
	  1. 客户端发起与mongos的链接
	  
	  2. Mongos服务端accept接收链接后，链接建立成功
	  
	  3. 客户端发送db.isMaster()命令给服务端
	  
	  4. 服务端应答isMaster给客户端
	  
	  5. 客户端发起与mongos代理的sasl认证(多次和mongos交互)
	  
	  6. 客户端发起正常的find()流程
	  
	  客户端SDK链接建立成功后发送db.isMaster()给服务端的目的是为了负载均衡策略和判断节点是什么类型，保证客户端快速感知到访问时延高的代理，从而快速剔除往返时延高的节点，同时确定访问的节点类型。
	  
	  此外，通过提前部署的脚本,该脚本在系统负载高的时候自动抓包，从抓包分析结果如下图所示：
	  
	  [![](https://mongoing.com/wp-content/uploads/2020/10/3c59dc048e88502.jpg)](https://mongoing.com/wp-content/uploads/2020/10/3c59dc048e88502.jpg)
	  
	  上图时序分析如下：
	  
	  1.  11:21:59.506174链接建立成功
	  
	  2.  11:21:59.506254 客户端发送db.IsMaster()到服务端
	  
	  3.  11:21:59.656479 客户端发送FIN断链请求
	  
	  4.  11:21:59.674717 服务端发送db.IsMaster()应答给客户端
	  
	  5.  11:21:59.675480 客户端直接RST
	  
	  第3和第1个报文之间相差大约150ms，最后和业务确定该客户端IP对应的超时时间配置，确定就是150ms。此外，其他抓包中有类似40ms、100ms等超时配置，通过对应客户端和业务确认，确定对应客户端业务接口超时时间配置的就是40ms、100ms等。因此，结合抓包和客户端配置，可以确定当代理超过指定超时时间还没有给客户端db.isMaster()返回值，则客户端立马超时，超时后立马发起重连请求。
	  
	  **总结：**通过抓包和mongos日志分析，可以确定链接建立后快速断开的原因是：客户端访问代理的第一个请求db.isMaster()超时了，因此引起客户端重连。重连后又开始获取db.isMaster()请求，由于负载CPU 100%, 很高，每次重连后的请求都会超时。其中配置超时时间为500ms的客户端，由于db.isMaster()不会超时，因此后续会走sasl认证流程。
	  
	  **因此可以看出，系统负载高和反复的建链断链有关，某一时刻客户端大量建立链接(2.2W)引起负载高，又因为客户端超时时间配置不一，超时时间配置得比较大得客户端最终会进入sasl流程，从内核态获取随机数，引起sy%负载高，sy%负载高又引起客户端超时，这样整个访问过程就成为一个“死循环”，最终引起mongos代理雪崩。** ([View Highlight](https://read.readwise.io/read/01j771yreypecbxxt5931m2atg))
	- **测试结果：**通过修改MongoDB内核版本故意让客户端超时反复建链断链，在linux-2.6版本中，1500以上的并发反复建链断链系统CPU sy% 100%的问题即可浮现。但是，在Linux-3.10版本中，并发到10000后，sy%负载逐步增加，并发越高sy%负载越高。
	  
	  **总结：**linux-2.6系统中，MongoDB只要每秒有几千的反复建链断链，系统sy%负载就会接近100%。Linux-3.10，并发20000反复建链断链的时候，sy%负载可以达到30%，随着客户端并发增加，sy%负载也相应的增加。Linux-3.10版本相比2.6版本针对反复建链断链的场景有很大的性能改善，但是不能解决根本问题。 ([View Highlight](https://read.readwise.io/read/01j774mwc7nndh8cm4f2pjtjdq))
	- **2.4 客户端反复建链断链引起sy% 100%根因** 
	  id:: 66df1e7c-7a8a-4c7b-996f-3a0b8bbff992
	  
	  为了分析%sy系统负载高的原因，安装perf获取系统top信息，发现所有CPU消耗在如下接口：
	  
	  [![](https://mongoing.com/wp-content/uploads/2020/10/1ff1de774005f8d.jpg)](https://mongoing.com/wp-content/uploads/2020/10/1ff1de774005f8d.jpg)
	  
	  从perf分析可以看出，cpu 消耗在_spin_lock_irqsave函数，继续分析内核态调用栈，得到如下堆栈信息：
	  
	  – 89.81% 89.81% [kernel] [k] _spin_lock_irqsave ▒
	  
	  – _spin_lock_irqsave ▒
	  
	  – mix_pool_bytes_extract ▒
	  
	  – extract_buf ▒
	  
	  extract_entropy_user ▒
	  
	  urandom_read ▒
	  
	  vfs_read ▒
	  
	  sys_read ▒
	  
	  system_call_fastpath ▒
	  
	  0xe82d
	  
	  上面的堆栈信息说明，MongoDB在读取 /dev/urandom ，并且由于多个线程同时读取该文件，导致消耗在一把spinlock上。
	  
	  到这里问题进一步明朗了，故障root case 不是每秒几万的连接数导致sys 过高引起。根本原因是每个mongo客户端的新链接会导致MongoDB后端新建一个线程，该线程在某种情况下会调用urandom_read 去读取随机数/dev/urandom ，并且由于多个线程同时读取，导致内核态消耗在一把spinlock锁上，出现cpu 高的现象。 ([View Highlight](https://read.readwise.io/read/01j774q6zfhybk5w9wk8k6b2v8))
	- **为什么突发流量业务会抖动？**
	  
	  答：由于业务是java业务，采用链接池方式链接mongos代理，当有突发流量的时候，链接池会增加链接数来提升访问MongoDB的性能，这时候客户端就会新增链接，由于客户端众多，造成可能瞬间会有大量新连接和mongos建链。链接建立成功后开始做sasl认证，由于认证的第一步需要生成随机数，就需要访问操作系统”/dev/urandom”文件。又因为mongos代理模型是默认一个链接一个线程，所以会造成瞬间多个线程访问该文件，进而引起内核态sy%负载过高。 ([View Highlight](https://read.readwise.io/read/01j772mdkr74yrmm2snvxtwwz6))
	- **为什么数据节点没有任何慢日志，但是代理负载却CPU sy% 100%？**
	  
	  答：由于客户端java程序直接访问的是mongos代理，所以大量链接只发生在客户端和mongos之间，同时由于客户端超时时间设置太短(有接口设置位几十ms，有的接口设置位一百多ms，有的接口设置位500ms)，就造成在流量峰值的时候引起连锁反应(突发流量系统负载高引起客户端快速超时，超时后快速重连，进一步引起超时，无限死循环)。Mongos和mongod之间也是链接池模型，但是mongos作为客户端访问mongod存储节点的超时很长，默认都是秒级别，所以不会引起反复超时建链断链。 ([View Highlight](https://read.readwise.io/read/01j772nadz67f2dtjas0egc8jc))
	- **如果MongoDB集群采用普通复制集模式，客户端频繁建链断链是否可能引起mongod存储节点同样的****”雪崩”？**
	  
	  答：会。如果客户端过多，操作系统内核版本过低，同时超时时间配置过段，直接访问复制集的mongod存储节点，由于客户端和存储节点的认证过程和与mongos代理的认证过程一样，所以还是会触发引起频繁读取”/dev/urandom”文件，引起CPU sy%负载过高，极端情况下引起雪崩。 ([View Highlight](https://read.readwise.io/read/01j772q8xeyf2m7qy4mksv6p6y))