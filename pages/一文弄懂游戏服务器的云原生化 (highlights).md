title:: 一文弄懂游戏服务器的云原生化 (highlights)
author:: [[知乎专栏]]
full-title:: "一文弄懂游戏服务器的云原生化"
category:: #articles
url:: https://zhuanlan.zhihu.com/p/642960237
![](https://readwise-assets.s3.amazonaws.com/media/uploaded_book_covers/profile_1160846/v2-5206eeac8ce2f6b634c6b3050681cd55_720w.jpg)

- Highlights first synced by [[Readwise]] [[Jan 25th, 2024]]
	- 游戏服务器云原生化
	  
	  游戏服务器上云与传统互联网行业有着不同的特点，因此容器化改造需要一方面保留Kubernetes中适用的部分，另一方面摒弃掉不适合的部分并替换成新的实现。具体来说，要解决的重点问题如下：
	  
	  1.  为了降低延时，需要客户端直连游戏服，而非通过暴露Service的方式做负载均衡。
	  2.  游戏服是有状态的，缩容时不能直接停掉服务器，会导致玩家掉线；而是要等到这个服没有玩家后再停止服务。
	  
	  具体上云的实现逻辑，行业内并没有统一的框架或规范，游戏厂商各家有各家的实现。目前较知名的框架有Agones、KruiseGame，各自的侧重点也不尽相同。我在学习过程中调研过以下几个开源项目，分别做下介绍。
	  
	  ![](https://pic2.zhimg.com/80/v2-bb11a2e1e3541b888913daa658260bfd_720w.webp)
	  
	  [这个项目](https://link.zhihu.com/?target=https%3A//github.com/markmandel/paddle-soccer)在2016年推出，是一个实验性质的项目，用于验证游戏服务器在Kubernetes上的管理。这个项目的发起人Mark Mandel来自谷歌，他同时也是Agones项目的发起人。可以说，Agones就是在这个项目的基础上发展和完善的。
	  
	  这个项目的目的是通过一个简单的足球对战游戏，来验证游戏服在Kubernetes上的托管。架构上分为匹配服和游戏服，弹性扩容主要是针对后者。作者为这个项目写过[四篇博文](https://link.zhihu.com/?target=https%3A//blog.csdn.net/Jailman/article/details/127259413)，详细介绍了项目目的和设计思路。此外，作者还在GDC 2017上做过[专题演讲](https://link.zhihu.com/?target=https%3A//www.gdcvault.com/play/1024328/)。
	  
	  流程是客户端通过匹配服查找要连接到的游戏服，然后将游戏服的ip和端口返回给客户端，客户端连接到游戏服，开始比赛。这里为了延时考虑，客户端直连游戏服，而非像传统互联网应用一样通过Service做负载均衡。实现方法是将游戏服Pod的hostNetwork设置为true，这样Pod就可以直接使用宿主的网络名称空间，外部连接没有额外延迟。
	  
	  由于各个Pod共用宿主机的网络空间，那么就可能出现端口冲突。解决办法是将已使用的端口记录到Redis中，新服务器启动时先向Redis获取已使用的端口并排除掉，然后在剩余的端口范围中随机，直到获取到可用的端口。
	  
	  由于游戏服是有状态的，因此不能使用无状态的Deployment来管理，而是通过Pod加Label的方式自行实现scaler。由Label来区分服务器是何种类型（匹配服还是游戏服），并将同种类型的服务器安排在同一节点上。
	  
	  在扩缩容上，这个项目优先考虑节点的扩缩容而非游戏服，因为节点数量与成本直接相关。为集群设置最大节点数和最小节点数，并为每个节点设置CPU缓冲区大小，避免耗尽节点的CPU。
	  
	  为了避免集群碎片化，利用节点亲和性，让游戏服务器尽量集中到已有节点中。当某个节点使用的CPU降低到一定阈值时，封锁节点，不再允许调度新的Pod到这个节点上。当这个节点上的所有游戏服都退出后，再删除这个节点。
	  
	  paddle-soccer在代码中与谷歌云强绑定，所以只能在谷歌云中使用。如果想在别的云平台使用，除非自己修改代码。
	  
	  ![](https://pic3.zhimg.com/80/v2-ad486201c9b8b5ecb31ca01b1b273b46_720w.webp)
	  
	  [Agones](https://link.zhihu.com/?target=https%3A//github.com/googleforgames/agones)在2017年推出，是谷歌和育碧联合发起的游戏服务器云原生解决方案。前面说到，它的发起人和paddle-soccer是同一人，但是它在后者的基础上做了进一步的封装和抽象，并引入育碧在游戏行业开发3A的经验，使得通用性更强、更容易使用。
	  
	  与paddle-soccer类似，Agones对于游戏服务器采用了直连的方式，而非通过Service做负载均衡。
	  
	  对于扩缩容这块，首先需要理解Agones的工作模型。游戏服通过Fleet来管理。Fleet上维护有所有可用的游戏服，这些游戏服可能处于Allocated状态或Ready状态。Allocated状态代表已投入使用的服务器，通过匹配服务可以分配到；而Ready状态代表已准备好，但还没投入使用的服务器。扩容和缩容都是针对Fleet。扩容时向Fleet添加新的状态为Ready的服务器，缩容时仅从Fleet删除状态为Ready的服务器，这样能保护正在使用的服务器不被异常关闭。如果确想关闭状态为Allocated的服务器，可以先由业务端调用SDK将服务器状态置为Ready。
	  
	  另一个关键概念是FleetAutoScaler，它用来做弹性扩缩容。可以通过FleetAutoScaler设置Fleet的最大和最小副本数，以及缓冲区大小。缓冲区也就是Ready服务器的数量。通过这些设置，Fleet的服务器数量会控制在最大和最小值之间，而且扩缩容时会保持固定的缓冲区大小。
	  
	  通过Fleet和FleetAutoScaler，这个模型维护一组预热的游戏服，可随时根据需要投入使用，避免硬启动带来的需求满足延迟。
	  
	  Agones还提供了一套完善的SDK供业务端使用。SDK目前支持Go、C++、Node.js等多种语言，以及Ready()、ShutDown()等查询或者设置游戏服状态的API。业务端调用SDK时，会向SDK服务器发送gRpc请求，由SDK服务器负责处理和返回。
	  
	  除此之外，Agones还提供健康检查的功能。当判断游戏服处于不健康状态时，处于Fleet管理下的游戏服会被删除并重新创建。
	  
	  Agones可以在各种云平台和Minikube中使用。但是在Mac系统上跑Minikube时，有可能出现从外部连不上游戏服务器的问题，这个目前还没有太好的解决办法。
	- [[OpenKruiseGame]]
		- ![](https://pic3.zhimg.com/80/v2-6d8f27340225a6d456d810805fb6dcda_720w.webp)
		  id:: 65b1fb4f-3578-40b6-bb67-2607997b467e
		  
		  [OpenKruiseGame](https://link.zhihu.com/?target=https%3A//github.com/openkruise/kruise-game)（OKG）由阿里云于2022年推出，目前还是一个比较新的项目。它旨在帮助业务开发者简化游戏云原生化的过程，并提供了热更新、原地升级和定向管理等实用功能。
		  
		  OKG属于[[OpenKruise]]的子项目，前者也依赖后者来构建。OpenKruise是阿里云推出的一个Kubernetes扩展功能集，它提供了Kubernetes原生所没有的一些功能，如：通用工作负载、原地升级、可配置扩缩容等。OpenKruise目前已在互联网行业得到广泛应用，如阿里巴巴、斗鱼、Boss直聘、小红书等。
	-
	- 与[[Agones]]目标对象局限于PVP游戏不同， [[OpenKruiseGame]] 的目标对象不仅包括[[PVP]]游戏，也包括[[PVE]]游戏。**这使得OKG在功能设计上与Agones有所不同，除了传统的弹性扩缩容，还包括热更新及定向运维管理等功能，这些功能在PVE游戏中更为需要。** #card
	  id:: 65b1f9df-b01f-40d4-88b4-17ba4258b21c
		- 与PVP游戏相比，PVE类型的游戏特点是用户状态保存的时间更长，不像有明确对局时间的开房间游戏（PVP）那样一局只有十几分钟，理论上PVE游戏中用户只要愿意可以一直在线，这个过程中状态需要一直维持。
		- 为此，**OKG首先添加了定向运维的功能。通过设置要定向管理（Reserve）的服务器id，删除对应id的游戏服，同时在创建新服时避免该id对应的游戏服生成。**在缩容时，OKG优先删除要Reserve的游戏服，然后依次按照状态、优先级和服务器id等参数确定剩余游戏服的删除顺序。通过Reserve指定下线的功能，适合处理PVE游戏经常会遇到的合服场景。
		- 其次，**OKG提供了热更新的功能**。很多语言（如Go）不具备语言级别的热更新能力，但可以通过容器的方式支持。OKG的热更新功能由底层的OpenKruise提供。**实现方法是在每个游戏服Pod中部署Sidecar和Main两个容器，Sidecar放游戏脚本，Main放游戏引擎和守护进程。热更新时只更新Sidercar，Main不动。这样就无需对Pod整体重建，从而做到在玩家不中断游戏的前提下修改游戏逻辑。**
	- 与Agones类似，OKG使用CRD来自定义游戏服相关的工作负载，包括两种：GameServerSet与GameServer。GameServerSet是对一组游戏服管理的抽象，类比于Agones中的Fleet，它是对OpenKruise中的Advanced StatefulSet的进一步扩展，因此也具有后者特有的原地升级（热更新）功能。GameServer是对一个游戏服管理的抽象，也就是对游戏服定向运维动作的记录，因此删除GameServer也不会触发实际游戏服的删除。
	- 截止本文写作时止，OKG刚刚迭代到0.4版本。作为一个新兴的项目，前面还有很长的路要走。 ([View Highlight](https://read.readwise.io/read/01hmzkagfr40dvh9mcyk4bzh3m))