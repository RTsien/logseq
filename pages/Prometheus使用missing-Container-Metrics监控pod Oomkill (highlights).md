title:: Prometheus使用missing-Container-Metrics监控pod Oomkill (highlights)
author:: [[Mister Li]]
full-title:: "Prometheus使用missing-Container-Metrics监控pod Oomkill"
category:: #articles
url:: http://www.lishuai.fun/2021/07/06/prometheus-create-missing-containerer-mitrics/
![](https://hugo-doc-img.oss-cn-shanghai.aliyuncs.com/img/image-20210706113837427.png)

- Highlights first synced by [[Readwise]] [[Dec 5th, 2023]]
	- Kubernetes 默认情况下使用 cAdvisor 来收集容器的各项指标，足以满足大多数人的需求，但还是有所欠缺，比如缺少对以下几个指标的收集：
	  
	  missing-container-metrics 这个项目弥补了 cAdvisor 的缺陷，新增了以上几个指标，集群管理员可以利用这些指标迅速定位某些故障。例如，假设某个容器有多个子进程，其中某个子进程被 OOM kill，但容器还在运行，如果不对 OOM kill 进行监控，管理员很难对故障进行定位。 ([View Highlight](https://read.readwise.io/read/01hgwvgg7awddw7jzpwqf19wa7)) #[[k8s]] #[[card]] #[[oom]]
		- 💡: https://github.com/draganm/missing-container-metrics/blob/master/docker/event_handler.go#L65
		  通过监听containerd/docker的event来统计oom指标