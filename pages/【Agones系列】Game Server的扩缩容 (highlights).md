title:: 【Agones系列】Game Server的扩缩容 (highlights)
author:: [[知乎专栏]]
full-title:: "【Agones系列】Game Server的扩缩容"
category:: #articles
url:: https://zhuanlan.zhihu.com/p/537532777
summary:: This article is a tutorial on how to scale game servers using Agones, an open-source project for managing game infrastructure on Kubernetes. Agones introduces the concept of fleets and game server sets (GSS) to manage the scaling of game servers. The tutorial provides step-by-step instructions on how to create a fleet, manually scale the fleet, and set up automatic scaling using the FleetAutoscaler. It also explains the concept of buffer size and how it relates to the number of allocated game servers.
![](https://picx.zhimg.com/v2-d399b2e0e289f3ccdec0564f41681de0_720w.jpg?source=172ae18b)

- Highlights first synced by [[Readwise]] [[Jun 20th, 2025]]
	- 这里通过fleetName指定自动扩缩绑定的Fleet对象。自动扩缩类型为buffer，buffer大小为2，最大的副本数指定为10。这里的buffer大小是相对于Allocated状态gs的个数。我们知道，正常创建的gs会处于Ready状态，Agones为适应open Match匹配机制设计了一种Allocated状态，代表分配使用，处于此状态的gs意味着游戏服可以被玩家连接进行游戏。buffer大小为2则表示gs的副本数量要比Allocated的gs数量多2。可以看出，buffer存在的意义是在玩家数量徒增时可以让fleet自动扩容，进而减少游戏服启动的时间。我们来配合gameserverallocation使用一下FleetAutoscaler。
	  
	  首先创建一个gameserverallocation
	  
	    apiVersion: "allocation.agones.dev/v1"
	    kind: GameServerAllocation
	    spec:
	      required:
	        matchLabels:
	          agones.dev/fleet: simple-game-server
	  
	  这里使用matchLabels匹配到对应的fleet
	  
	  此时所有gs的状态如下所示，有一个gs的状态变为Allocated
	  
	    kubectl get gs
	    NAME                             STATE       ADDRESS           PORT   NODE                     AGE
	    simple-game-server-rhjtz-g72pq   Ready       xxx.xxx.xxx.xxx   7483   cn-xxx.xxx.xxx.xxx.xxx   145m
	    simple-game-server-rhjtz-gnhnx   Ready       xxx.xxx.xxx.xxx   7012   cn-xxx.xxx.xxx.xxx.xxx   147m
	    simple-game-server-rhjtz-jqpxt   Ready       xxx.xxx.xxx.xxx   7768   cn-xxx.xxx.xxx.xxx.xxx   145m
	    simple-game-server-rhjtz-pstgv   Ready       xxx.xxx.xxx.xxx   7486   cn-xxx.xxx.xxx.xxx.xxx   145m
	    simple-game-server-rhjtz-zhxts   Allocated   xxx.xxx.xxx.xxx   7909   cn-xxx.xxx.xxx.xxx.xxx   147m
	  
	  我们现在进行缩容，调整gs数目为0
	  
	    kubectl get gs
	    NAME                             STATE       ADDRESS           PORT   NODE                     AGE
	    simple-game-server-rhjtz-zhxts   Allocated   xxx.xxx.xxx.xxx   7909   cn-xxx.xxx.xxx.xxx.xxx   147m
	  
	  看到虽然副本数为0，但是依然Allocated状态的gs是不会被删除的，因为此时游戏可能在进行中。 ([View Highlight](https://read.readwise.io/read/01jy4dnnkyt3ban97ffaymbjr8))