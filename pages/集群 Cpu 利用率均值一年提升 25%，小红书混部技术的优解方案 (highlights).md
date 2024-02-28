title:: 集群 Cpu 利用率均值一年提升 25%，小红书混部技术的优解方案 (highlights)
author:: [[等你加入的]]
full-title:: "集群 Cpu 利用率均值一年提升 25%，小红书混部技术的优解方案"
category:: #articles
url:: https://mp.weixin.qq.com/s/tM3X42N_0S7sKY4BOMEwVg
summary:: “资源利用率” 和 “系统稳定性” 的平衡追求
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/YriaiaJPb26VMpALJwKDLXBc8QicvIEoebayvcvyuAuGA1trsSkLhDRfmT8L0WCUQgFuFumtaQmKnibOKKa7huhIQw/0?wx_fmt=jpeg)

- Highlights first synced by [[Readwise]] [[Feb 4th, 2024]]
	- **整体架构上**，离线业务发布入口统一收敛在在一个集群，我们称之为元数据集群，目的是为业务屏蔽底层多物理 K8s 集群。通过 Virtual-Kubelet 连接元数据集群与物理集群，将闲置资源汇聚到元数据集群，在元数据集群中调度分发转码类任务到底层物理集群。
	  
	  **策略方面**，二次调度器负责巡检集群中的所有节点，识别出低效节点并进行标记；随后 Virtual-Kubelet 获取物理集群中的低效节点可用资源作为集群闲置资源，再次分配给离线转码场景。同时，二次调度器确保一旦在线服务有资源需求，将会立刻驱逐离线 Pod 并归还资源。通过此举，我们能够提高集群资源的利用效率，减少资源浪费，并满足转码类场景对计算资源的需求。 ([View Highlight](https://read.readwise.io/read/01hnky835x7hdgrpb7eay6wxwn))