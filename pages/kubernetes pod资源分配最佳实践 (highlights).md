title:: kubernetes pod资源分配最佳实践 (highlights)
author:: [[running-in-sky]]
full-title:: "kubernetes pod资源分配最佳实践"
category:: #articles
url:: https://www.zzzhy.cn/2020/01/08/kubernetes-resource-best-practice/
summary:: 一、服务质量等级 QoS Guaranteed pod内所有容器的cpu和内存都配置了request和limit的限额，并且都各自相等  Burstable 至少一个容器配置了，且不满足Guaranteed要求  BestEffort Pod 里的容器必须没有任何内存或者 CPU的限额配置   二、驱逐 与 OOM_Killer（内存相关）因为 kubelet 默认每 10 秒抓取一次 cAdvi
![](https://readwise-assets.s3.amazonaws.com/static/images/article2.74d541386bbf.png)

- Highlights first synced by [[Readwise]] [[Apr 11th, 2024]]
	- 因为 kubelet 默认每 `10` 秒抓取一次 cAdvisor 的监控数据，所以有可能在 kubelet 驱逐 Pod 回收内存之前发生内存使用量激增的情况，这时就有可能触发内核 OOM killer。这时删除容器的权利就由kubelet 转交到内核 OOM killer 手里，但 kubelet 仍然会起到一定的决定作用，它会根据 Pod 的 **QoS** 来设置其 `oom_score_adj` 值 ([View Highlight](https://read.readwise.io/read/01hv41qmx7w6nhvy24hhyzwde7))
	- 因为`DaemonSet` 的Pod不应被驱逐，应确保`DaemonSet`的Pod声明为Guaranteed的QoS ([View Highlight](https://read.readwise.io/read/01hv41v1kqhph0fjdeykk06mq6))