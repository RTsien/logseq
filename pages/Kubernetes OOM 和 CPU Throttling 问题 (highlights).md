title:: Kubernetes OOM 和 CPU Throttling 问题 (highlights)
author:: [[flashcatcloud]]
full-title:: "Kubernetes OOM 和 CPU Throttling 问题"
category:: #articles
url:: https://flashcat.cloud/blog/troubleshoot-kubernetes-oom/
![](https://readwise-assets.s3.amazonaws.com/static/images/article3.5c705a01b476.png)

- Highlights first synced by [[Readwise]] [[Dec 5th, 2023]]
	- 监控 Kubernetes CPU throttling
	  
	  您可以检查进程与 Kubernetes limits 的接近程度：
	  
	    (sum by (namespace,pod,container)(rate(container_cpu_usage_seconds_total
	    {container!=""}[5m])) / sum by (namespace,pod,container)
	    (kube_pod_container_resource_limits{resource="cpu"}))
	    
	  
	  如果我们想要跟踪集群中发生的限制量，cadvisor 提供了 container_cpu_cfs_throttled_periods_total 和 container_cpu_cfs_periods_total 两个指标。通过这两个指标，您可以轻松计算所有 CPU 周期内的限制百分比。 ([View Highlight](https://read.readwise.io/read/01hgww2hp5ga3b96rvt4qms48v))