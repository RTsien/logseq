title:: TDSQL 3.0的架构设计 (highlights)
author:: [[woa.com]]
full-title:: "TDSQL 3.0的架构设计"
category:: #articles
url:: https://iwiki.woa.com/p/4010269138
summary:: TDSQL 3.0 is designed to improve upon its predecessor by addressing issues like syntax compatibility, flexibility, and complex queries while maintaining the advantages of a sharded architecture. It features a three-layer storage model and a peer-to-peer architecture that enhances performance and scalability. Additionally, TDSQL 3.0 supports online DDL operations and parallel query execution to better handle large datasets and improve efficiency.
![](https://readwise-assets.s3.amazonaws.com/static/images/article0.00998d930354.png)

- Highlights first synced by [[Readwise]] [[May 13th, 2025]]
	- **对等架构**
	  
	  ![](https://iwiki.woa.com/tencent/api/attachments/s3/url?attachmentid=19278832)
	  
	  从广义的概念来说，分布式数据库既可以包括Shared-Storage的云原生架构，也可以包括Shared-Nothing的分布式架构，当然也还可以包括Shared-Storage+Shared-Nothing的混合分布式架构。
	  
	  > *Shared-Nothing和Shared-Storage区别：数据处理是否有Owner，某一行数据只能某一个节点处理，那就是Shared-Nothing；如果任意节点可以处理任意数据，那就是Shared-Storage或Shared-Everything。*
	  
	  **Shared-Storage**
	  
	  代表产品有AWS Aurora，阿里云PolarDB，腾讯云TDSQL-C，Google AlloyDB等。
	  
	  ①：核心优势是存储按需分配和计费，且可以扩展至100T级别；因为主节点有Buffer Pool的存在，如果数据缓存命中率非常高的情况下，单核性能表现非常好。
	  
	  ②：单主架构，写扩展性不足，虽然目前各个厂商都在攻坚或发布多写架构（Multi-Master），但是类似于AWS的Multi-Master更多是解决高可用问题，而PolarDB的则更多是类似数据进行Sharding的方案，本质上还是单节点写。从发展来看，其架构会越来越趋近于Oracle的RAC架构，实现Cache Fusion，但工程量大难度高，而且RAC架构的扩展性也存在上限，Oracle RAC白皮书中虽然可以支持128个节点，但是在实际投产过程中，基本大部分在4个节点以下。
	  
	  **Shared-Nothing**
	  
	  代表产品有Google Spanner、蚂蚁金服OceanBase、PingCAP的TiDB等，但是各自架构又有一些差异。
	  
	  ①：架构优势：多写多读，横向扩展性友好（尤其是业务具备一定水平拆分能力的场景）。
	  
	  ②：架构劣势：跨节点分布式事务、数据远程网络访问等带来单核性能和延迟问题。
	  
	  **TiDB：**
	  
	  ①：计算与存储完全分离，计算层对数据的访问通过网络KV协议访问
	  
	  ②：为了确保多个SQLEngine均可以提供读写服务，所以计算层没有数据缓存（不过TiDB实现了“[缓存表](https://docs.pingcap.com/zh/tidb/stable/cached-tables)”“[下推计算结果缓存](https://docs.pingcap.com/zh/tidb/stable/coprocessor-cache)”等读缓存机制，对部分读场景提速）
	  
	  ③：TiDB本身还有另外一个问题，就是计算层使用Go语言，存储层采用Rust语言，当需要做计算下推的时候，需要把计算逻辑和数据schema进行下推，这些在两层需要各实现一次，导致重复工作量与代码一致性等问题。
	  
	  **OceanBase：**
	  
	  ①：每个节点里同时包含计算和存储功能；每个节点负责一部分数据（tablet）的存储。
	  
	  ②：OB的设计原则里，尽可能让计算访问本地数据（ OBProxy 将请求转发到存储的主节点，防止SQL发到备机生成远程计划，从而避免的额外RPC访问）
	  
	  ③：OB的节点类型包括全能型副本（可读可写），只读型副本（不参与选举，异步副本，提供弱一致性读服务），日志副本（同步副本，仅保留日志，不能被选举为主，不能提供任何读写服务）
	  
	  **TDSQL 3.0：**
	  
	  ![](https://iwiki.woa.com/tencent/api/attachments/s3/url?attachmentid=19278854)
	  
	  在3.0的架构设计中，我们既要让计算离存储（缓存/存储）尽可能的近，也要能做到计算与存储分离实现高弹性，因此我们设计为：
	  
	  ①：采用对等架构设计，每个节点（进程）均包含有完整的计算、存储、日志三个功能引擎。
	  
	  ②：在实际部署中，通过元数据及调度模块，根据不同业务场景的需要，对节点功能进行划分，例如支持全功能节点（计算存储日志三个服务都提供），计算节点（仅支持计算，所有数据通过Remote访问），存储节点（仅支持数据存储），日志服务（仅提供日志订阅服务）等。
	  
	  ③：计算层与本地存储采用本地访问模式，访问远端存储则采用网络RPC访问模式，尽可能的确保访问本地数据的性能。 ([View Highlight](https://read.readwise.io/read/01jv3yp5zt1d1yjxvmy1rqbzf3))
	- **某业务场景的分库分表与二级索引场景**
	  
	  **业务场景：**因为访问量比较大，业务自行做了分库分表，由8个MySQL实例，根据用户ID（FUId） hash分库分表，但是现在部分新业务场景需要根据业务ID（FBId）进行查询，这种情况下，有两种解决方案：
	  
	  1、 直接对8个MySQL实例分别查询后再对结果进行合并，这个会导致查询性能较低，且异常处理困难；
	  
	  2、 直接新建一张索引表，根据FBId先查询到对应的FUId后，再到对应的MySQL实例进行查询，改方案要求在对数据更新时，同时更新数据表和索引表，且二者保障一致。
	  
	  以上两个方案均存在两次网络交付带来的延迟问题，以及各种故障异常情况下带来的数据一致性等问题，对业务使用很不友好。
	  
	  **解决方案：**TDSQL 3.0支持二级索引，建表语句如下即可：
	  
	  **主要收益：**从原理上来说，3.0内部自建了一个索引表，根据二级索引查询，也需要先查询一次索引，再根据主键查询详细数据，但是带来以下几个好处：
	  
	  1、 访问路径更短，时延更低；
	  
	  2、 数据强一致；
	  
	  3、 业务无需多次写入或多个节点查询。 ([View Highlight](https://read.readwise.io/read/01jv40krrv9nw0z5kr6fegc618))