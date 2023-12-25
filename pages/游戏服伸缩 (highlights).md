title:: 游戏服伸缩 (highlights)
author:: [[openkruise.io]]
full-title:: "游戏服伸缩"
category:: #articles
url:: https://openkruise.io/zh/kruisegame/user-manuals/gameservers-scale/#openkruisegame%E7%9A%84%E6%B0%B4%E5%B9%B3%E4%BC%B8%E7%BC%A9%E7%89%B9%E6%80%A7

- Highlights first synced by [[Readwise]] [[Dec 11th, 2023]]
	- 游戏服与无状态业务类型不同，对于自动伸缩特性有着更高的要求，其要求主要体现在缩容方面。
	  
	  由于游戏为强有状态业务，随着时间的推移，游戏服之间的差异性愈加明显，缩容的精确度要求极高，粗糙的缩容机制容易造成玩家断线等负面影响，给业务造成巨大损失。
	  
	  原生Kubernetes中的水平伸缩机制如下图所示
	  
	  ![](https://openkruise.io/zh/assets/images/autoscaling-k8s-2f9fa58833b5fddd2ca762cd9c7d58e7.png)
	  
	  在游戏场景下，它的主要问题在于：
	  
	  •   在pod层面，无法感知游戏服业务状态，进而无法通过业务状态设置删除优先级
	  •   在workload层面，无法根据业务状态选择缩容对象
	  •   在autoscaler层面，无法定向感知游戏服业务状态计算合适的副本数目
	  
	  这样一来，基于原生Kubernetes的自动伸缩机制将在游戏场景下造成两大问题：
	  
	  •   缩容数目不精确。容易删除过多或过少的游戏服。
	  •   缩容对象不精确。容易删除业务负载水平高的游戏服。
	  
	  OKG 的自动伸缩机制如下所示
	  
	  ![](https://openkruise.io/zh/assets/images/autoscaling-okg-a45f816714e854f821386482be84e1b6.png)
	  
	  •   在游戏服层面，每个游戏服可以上报自身状态，通过自定义服务质量或外部组件来暴露自身是否为WaitToBeDeleted状态。
	  •   在workload层面，GameServerSet可根据游戏服上报的业务状态来决定缩容的对象，如[游戏服水平伸缩](https://openkruise.io/zh/kruisegame/user-manuals/gameservers-scale)中所述，WaitToBeDeleted的游戏服是删除优先级最高的游戏服，缩容时最优先删除。
	  •   在autoscaler层面，精准计算WaitToBeDeleted的游戏服个数，将其作为缩容数量，不会造成误删的情况。
	  
	  如此一来，OKG的自动伸缩器在缩容窗口期内只会删除处于WaitToBeDeleted状态的游戏服，真正做到定向缩容、精准缩容。 ([View Highlight](https://read.readwise.io/read/01hhcbeq5x9h7qxtw09gz0s22b))