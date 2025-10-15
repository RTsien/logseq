title:: 一人之下手游服务器架构设计 (highlights)
author:: [[woa.com]]
full-title:: "一人之下手游服务器架构设计"
category:: #articles
url:: https://km.woa.com/articles/show/462649
summary:: The text discusses the server architecture design of the mobile game "一人之下." The game uses a server structure with separate zones for different platforms and regions, such as QQ and WX. It also explains how player data is managed and cached for efficient gameplay.
![](https://readwise-assets.s3.amazonaws.com/static/images/article0.00998d930354.png)

- Highlights first synced by [[Readwise]] [[Mar 28th, 2024]]
	- matchsvr
	  
	  matchsvr负责不同的PVE，PVP玩法战斗匹配，matchsvr目前支持单个玩家匹配后创建单局，多人组队的方式进入创建单局。matchsvr除了匹配的功能外，就是负责单局战斗的创建，包括PVP和PVE，matchsvr维护了所有pvpgamesvr的状态和负载情况，以选择合适的pvpgamesvr进行单局的管理。
	  
	  目前针对有限有状态的matchsvr设计成单个大区一主一备，当主挂了之后，由备机提供服务。由于matchsvr上保存的是匹配数据，在服务挂了后，只影响当前玩家的匹配，表现就是匹配超时，切换到备机后，玩家只需要再次点击匹配就可以正常进行匹配操作。 ([View Highlight](https://read.readwise.io/read/01ht24dcmtge7bjftd4cn5gqa1))
		- 💡: 状态数据生命周期短
	- infosvr
	  
	  作为游戏比较重要的一个公共的服务，提供游戏内玩家之间基本信息的查询，游戏内关于玩家简要信息的展示，例如排行榜，场景玩家信息查看等都会通过infosvr进行查询。设计上infosvr作为无状态的服务，大区公用。
	  
	  因为infosvr会回写玩家简要数据，所以为了能让同一个角色的请求转发到同一个infosvr进行串行操作，需要通过角色id进行一致性HASH，除此之外还需要结合DB的data version来进行最终的数据修改操作以防止infosvr服务节点发生变更时，hash转发规则会发生变化，导致最终的并发修改。 ([View Highlight](https://read.readwise.io/read/01ht24mn2g8tzrx50ehprg7h3k))
		- 💡: db的data version如何生成的，操作请求里是否有带data_version
	- 、 ([View Highlight](https://read.readwise.io/read/01ht25252m1d8dcht5k100607m))
	- 玩家数据
	  
	  游戏中玩家的数据由tcaplus公共服务进行cache，并负责容灾和落地。tcaplus中的blob数据为了方便的扩展和动态更新，采用protobuf结构进行存储和解析。
	  
	  由于玩家的核心数据相对比较大，如果每次操作都从tcaplus进行拉取，反序列化，修改，序列化，回写，会有很大的性能开销和网络延迟，无法支撑大量玩家的并发操作。所以zonesvr上，会将登录的玩家数据在共享内存上进行cache，常用的cache方案有两种：
	  
	  •   cache的数据是序列化后的数据，每次操作会首先将本地的数据进行反序列化，然后进行修改，再序列化到本地。定时的针对玩家脏数据进行回写操作，保障数据落地到tcaplus。这样做的好处是：不用每次都远程操作tcaplus，只需操作本地cache的blob数据，且**版本更新对于数据修改可以支持热更，不需要进行停服**。缺点是即使每次操作只需要解析本地的blob数据，在玩家数据膨胀后，也会造成反序列化的性能开销变大，常用的优化方法是让protobuf支持懒解析的功能。
	  •   cache的数据是POD的数据，每次操作直接对玩家数据进行修改，不需要进行反序列化和序列化操作。没有任何性能开销。定时的针对玩家数据进行序列化回写操作，让数据落到tcaplus。这样做的好处是：玩家数据的修改没有任何成本。但是如果**玩家数据的结构发生修改就必须进行停服更新**。
	  
	  一人之下手游采用的是第二种方案：玩家Cache在共享内存上的数据是POD数据。POD的数据的结构是根据protobuf的proto IDL生成的，生成方式其实就是将message映射成struct，repeated结构生成一个IDL描述的固定长度的数组，保证protobuf的message能够变成一个共享内存上的POD结构。
	  
	  对于玩家cache的POD数据，有一个很大的**风险点**是：代码逻辑可能导致玩家的数据被写坏。因为玩家数据都是cache在共享内存上，且相互临近，所以如果代码逻辑有问题，可能导致玩家数据被写坏，且有可能无法察觉导致问题扩散。即使后来我们在玩家数据前后加了4K的保护页，但是针对这种POD的数据还是无法做到很好的保护。 ([View Highlight](https://read.readwise.io/read/01ht25420vmcydgwp9j02pj6z5))
	- 服务容灾
	  
	  在架构中，除了核心游戏服务器外，很多公共无状态的服务考虑性能和单点问题，需要部署多台，这里需要对这些无状态的公共服务进行容灾处理，一人之下通过zookeeper进行容灾处理。
	  
	  游戏中对zk的相关api调用封装在了zkagent中，需要容灾的服务会定时的通过zkagent上报自己的状态到zookeeper中，zkagent同样会定时的从zookeeper中拉取存活服务的最新状态列表，如果发现和本地的配置不一致则更新对应的配置并重启对应的服务。
	  
	  例如，zonesvr需要对router进行容灾，所有router会定时通过zkagent向zookeeper上报自己的状态，zonesvr的zkagent会定时的拉取所有router的状态，如果发现router集群的数量有发生变更，就用最新的所有router地址覆盖zonesvr本地的router配置，然后从新load zonesvr的路由配置。 ([View Highlight](https://read.readwise.io/read/01ht25jahjeft26a5y9bgqbddh))
		- 💡: [[red如何进行服务容灾]]]？
	- 路由不一致问题
	  
	  对于现有的架构，虽然通过zookeeper对无状态的服务进行了容灾，但是其实是有问题的，可能会存在路由不一致性的问题。例如zonesvr对router进行了容灾，当某台router挂掉之后，zookeeper更新了最新的router集群的状态。但是某个zonesvr可能此时到zookeeper的网络出现问题，无法拉取到router集群的最新状态，此时**这个zonesvr和其他zonesvr所看到的router列表就是不一致的**。  
	  ![](https://km.woa.com/asset/f6357287360d47c0b8a43fe6fd315943?height=445&width=839)
	  
	  假如某台router到zookeeper的网络不通，会导致router的上报出现问题，zookeeper中该router会失效，但是router到其他服务的网络都是正常的，这时候正常的router就会被从集群中剔除，不提供服务。虽然这对于无状态的router影响不是很大，但是这个问题还是存在的。假如zookeeper集群的网络出现问题，导致小部分router上报有效，大部分router的上报丢失，整个服务集群中，router的状态就会有很大的不一致。
	  
	  **同样router对于公共服务的容灾，也会存在各个router可能对公共服务的路由发生不一致的问题**。
	  
	  上面描述的问题，**发生的概率比较小，且发生了一般会段时间修复**，**如果出现了极限情况，长时间路由不一致，这个时候就需要手动进行干预处理**。 ([View Highlight](https://read.readwise.io/read/01ht25m7r2wjfy3ncffkztk0gg))
	- 单服支撑上限
	  
	  由于一人采用分区分服的架构设计，且一个游戏逻辑区只能在一个物理区上。所以导致了单服是有PCU和注册上限的限制。由于游戏中包含了主城的场景同步，目前单服PCU上限外网设置为1W。这就导致了不删档开服期间，开服的速度会非常的快，然后后面新开的服由于流失的原因很快就会变成鬼服，严重影响游戏的体验。
	  
	  其实可以借鉴现在大部分单句游戏的架构设计，采用全区全服的思想来设计分区分服的游戏。游戏的一个逻辑区可以分布在多个物理区，一个物理区承载多个逻辑区，这样游戏**逻辑区的PCU完全取决于配置和服务器的数量**。
	  
	  这样也会带来一些不方便地方就是: 作为RPG类型的游戏，很多单服的玩法也要通过跨服的方式进行实现，工作量上势必会增加很多。例如单服内的场景同步就需要采用跨服的设计方式将分布在不同物理服务器的玩家映射到一台单独的场景服务进行场景内的交互。 ([View Highlight](https://read.readwise.io/read/01ht25qtn6zcy6e9cw07hw009a))
	- 路由一致性和调度管理
	  
	  前面也分析了现有游戏架构中路由不一致的问题，以及对于有状态服务的路由均匀问题。对于这两个问题都需要引入一个协调者进行路由的管理，这个也是接下来我打算做的事情之一。
	  
	  对于路由一致性，协调者需要根据服务中心最新的服务状态，通过一致性算法(2PC, 3PC, TCC,raft, gossip等)进行各个router一致性路由的维护，以保证各个服务看到的路由信息是一致的，虽然可能不是最新。可用性对于业务来说还是很重要的。
	  
	  对于路由的调度，现有的一致性算法在请求节点key少量的情况下其实很难做到路由均衡的效果，特别是游戏服务中按zoneid一致性hash，如果是对于有状态服务，采用此种路由，势必会导致不均衡的后果。如果希望动态的进行有状态服务的路由就需要引入一个调度管理的角色。  
	  同样对于部分有状态服务的恢复也可以通过引入调度中心来进行及时的恢复，而不是现在借助与腾讯云的迁移技术进行业务的恢复。 ([View Highlight](https://read.readwise.io/read/01ht25sw51s6h61d2zzvtqfn9r))