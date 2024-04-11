title:: FinOps时代如何玩转应用资源配置 (highlights)
author:: [[Crane]]
full-title:: "FinOps时代如何玩转应用资源配置"
category:: #articles
url:: https://gocrane.io/zh-cn/docs/best-practices/how-to-optimize-your-application-resource/
![](https://readwise-assets.s3.amazonaws.com/static/images/article4.6bc1851654a0.png)

- Highlights first synced by [[Readwise]] [[Apr 11th, 2024]]
	- 加拿大软件公司 Densify 在《12 RISK OF KUBERNETES RESOURCE MANAGEMENT》[1]中总结了常见的资源配置问题。在下表中我们在它的基础上增加了副本数维度的分析。
	  
	  CPU Request
	  
	  Memory Request
	  
	  CPU Limit
	  
	  Memory Limit
	  
	  Replicas
	  
	  过大
	  
	  多余的CPU资源导致更多节点和资源的浪费
	  
	  调度器会申请过多Memory资源，导致更多节点和资源的浪费
	  
	  允许Pod申请过多的CPU资源从而产生“吵闹邻居”风险，影响同一节点上的其他Pod
	  
	  允许Pod申请过多的Memory资源从而产生“吵闹邻居”风险，从而影响同一节点上的其他Pod
	  
	  多余的Pod会导致更多节点和资源的浪费
	  
	  过小
	  
	  会导致在节点上过度堆叠Pod，如果所有CPU资源被用尽，则会在节点级别上产生争抢和CPU throttling的风险
	  
	  会导致在节点上过度堆叠Pod，如果所有Memory资源都被用尽，则会在节点级别上产生Pod终止的风险（OOM Killer）
	  
	  会限制Pod的CPU使用，如果实际业务压力超过Limit，会导致CPU throttling和性能下降
	  
	  会限制Pod的Memory使用，如果实际业务压力超过Limit，会触发OOM Killer杀死进程
	  
	  过少的Pod会带来过高的利用率，引发诸如性能下降，OOM Killer等稳定性问题
	  
	  不设置
	  
	  调度器将不确定在集群中可以调度多少Pod，并且过度堆叠的Pod会产生显著的性能风险和不均匀的负载
	  
	  调度器将不确定在集群中可以调度多少Pod，从而产生过度堆叠和Pod被OOM Kill的风险
	  
	  Pod将不受约束，放大“吵闹邻居”效应，并产生CPU throttling的风险
	  
	  Pod将不受约束，放大了“吵闹邻居”风险，如果节点内存耗尽，可能会导致OOM Killer启动
	  
	  N/A
	  
	  大家可以发现资源设置过小会引发稳定性问题，而相比之下资源设置大一些“仅仅”会导致资源浪费，在业务快速发展时期这些浪费是可以接受的。这就是许多企业上云后资源利用率普遍偏低的主要原因。下图是一个应用的资源用量图表，该 Pod 的历史用量的峰值与它的申请量 Request 之间，有30%的资源浪费。
	  
	  ![Resource Waste](https://gocrane.io/images/resource-waste.jpg) ([View Highlight](https://read.readwise.io/read/01hv41ygbzsx76myapd296tcfc))