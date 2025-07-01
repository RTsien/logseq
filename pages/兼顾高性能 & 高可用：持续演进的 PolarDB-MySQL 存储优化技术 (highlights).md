title:: 兼顾高性能 & 高可用：持续演进的 PolarDB-MySQL 存储优化技术 (highlights)
author:: [[李薇]]
full-title:: "兼顾高性能 & 高可用：持续演进的 PolarDB-MySQL 存储优化技术"
category:: #articles
url:: http://mysql.taobao.org/monthly/monthly/2025/04/01/
summary:: 背景
![](https://readwise-assets.s3.amazonaws.com/static/images/article0.00998d930354.png)

- Highlights first synced by [[Readwise]] [[May 11th, 2025]]
	- 在InnoDB引擎中对性能影响最大的I/O为 Redo 写I/O 和 Page 读写 I/O。
	  
	  •   Redo 写I/O直接影响事务提交性能。在InnoDB存储引擎中，事务提交的持久性保障依赖于Redo日志的强制写（fsync）操作完成，该过程作为Write-Ahead Logging (WAL)机制的核心环节，Redo的写I/O直接影响事务提交性能。Buffer Pool的脏页管理机制对事务性能同样构成关键影响。数据库通过LRU算法维护缓冲池中的数据页，事务对数据页的修改会生成脏页。后台刷脏线程通过异步方式将脏页刷新到磁盘数据文件，若出现I/O延迟高导致脏页积聚，将引发以下连锁反应：首先，Checkpoint机制无法正常推进，导致Redo日志文件空间无法及时重用；其次，Redo日志缓冲区因缺乏可用空间而触发频繁的强制同步写入，最终形成”脏页堆积→Checkpoint停滞→Redo日志阻塞→事务提交延迟”的恶性循环。因此Redo 和Page的写I/O都对事务提交性能至关重要。
	    
	  •   Page Double Write保证原子性。在持久化存储介质中，不同存储设备对写入操作的原子性支持存在显著差异。例如，SSD支持4KB粒度的原子写入，而HDD仅保证512B原子性。InnoDB采用16KB页作为存储单元，当事务对页内数据进行局部修改时，若直接写入原始数据位置，可能因存储介质的原子性不足导致页结构损坏。具体而言，16KB页的写入若中途中断，将产生部分写入问题，破坏原有数据的完整性。InnoDB引入Doublewrite Buffer机制，通过将修改页的完整副本先持久化至预分配的双写区域，待该副本稳定落盘后，再异步将修改页写入原始位置。这种双写策略确保崩溃恢复时，可通过双写区域的完整页副本重建逻辑一致的数据内容。但双写的机制会增加写I/O的压力，进一步加大了I/O性能对性能的影响。
	    
	  •   Page 读I/O影响数据库查询修改。数据库系统中，数据Page的访问与解析是核心I/O操作的关键路径，事务提交更新Page也需要保证Page在Buffer Pool中。当执行页级操作时，系统从Buffer Pool中查找Page，若不在Buffer Pool未命中，则需触发读I/O，从存储设备中读取该页，当数据量越大时，Buffer Pool有限的空间命中的概率越低，产生读I/O越多，读I/O的性能也直接反馈在数据库查询性能中，也影响了事务的提交性能，因此Page读I/O在数据库OLTP场景下至关重要 ([View Highlight](https://read.readwise.io/read/01jtyznera46rk8azxjeswtepg))
	- PolarStore的I/O优化
	  
	  PolarFS是用户态文件系统，PolarStore提供逻辑卷。PolarFS深度和DB数据特点结合，为不同数据特点提供了高度定制化的优化，通过IO打标的方式将数据特点传递给底层的分布式存储。
	  
	  •   写路径上，PolarFS能够将一个文件条带化的分散在多个存储节点上，由PolarSwitch 服务经过高速RDMA网络发送到对应的leader上，leader通过parallal raft协议和follower同步数据。整体的延迟包括PolarFS到Polarswitch软件栈开销，Polarswitch经过RDMA发送到存储节点的网络延迟，存储节点leader将数据通过RDMA同步到follower中，同时将数据写入高速介质Optane/AliSCM中，将数据成功消息返回给Polarswitch再到PolarFS，以上即可完成一次IO。ChunkServer会后台异步的将数据从高速介质Optane刷新到NVMe中，Optane作为写Cache做写加速，未来还有更加快速的设备AliSCM，持久化内存设备做写Cache加速。原理如图所示，每个写IO持久化到Journal file中即可返回，异步的将数据刷新到NVMe中释放高速介质的空间。PolarStore软件和RDMA技术相结合，系统运行过程实现内存零拷贝。
	    
	  •   PolarStore提供请求均为原子写，InnoDB中无需DoubleWrite Buffer，消除数据崩溃一致性保证带来的double write开销。
	    
	  •   读路径上，PolarStore利用存储节点上内存，构建了一个数TB的弹性内存池，根据PolarFS的IO打标，将数据库中的Page，cache在内存中，读请求能够直接从内存中获取直接返回给DB，仅仅需要经过高速RDMA网络和内存访问的延迟，同时根据数据I/O打标、三副本中仅需保留一份数据，以及合理的Cache淘汰策略，能够将最重要的数据Cache在弹性内存池中，大幅度提升关键数据的Cache命中率。 ([View Highlight](https://read.readwise.io/read/01jtyzwwd3jspm2gbv2r476ty2))
	- PolarStore支持PolarDB实现自动选主。故障发生在计算节点时，基于polarstore提供的voting disk能力实现自动切主，内核自动切换角色。老RW故障后，数据库内核自治，无需管控参与，PolarStore提供抢锁的机制选主，故障秒级切换，自动恢复服务。传统主备MySQL服务需要管控参与，主备切换时需等待Slave上Binlog追平，修改Master和Slave角色，恢复时间长。 ([View Highlight](https://read.readwise.io/read/01jtz03deeet1dtypc6mbzgek0))