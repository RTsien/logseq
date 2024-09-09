title:: Pod-OOM故障监控 (highlights)
author:: [[Cnblogs.com]]
full-title:: "Pod-OOM故障监控"
category:: #articles
url:: https://www.cnblogs.com/sss4/p/17148369.html
summary:: 前言 K8s集群和Node宿主机之间的监控覆盖默认是断层的； 需要借助OpenTelemetry实现IasS层(主机)+PasS(K8s)+SasS(微服务层)&#160;日志和监控数据，实现可观测性； 可观测平台可以实现故障的快速定位； 故障分析 Pod因内存不足OOM，一般由以下2种原因导致 原
![](https://readwise-assets.s3.amazonaws.com/media/uploaded_book_covers/profile_1160846/1122865-20240404084553578-1091370169.png)

- Highlights first synced by [[Readwise]] [[Sep 10th, 2024]]
	- 如果1个Pod分配内存的速度太快，以至于Kubelet没有在默认的检查窗口(默认10s)中发现它；
	  
	  Pod内存使用试图超过可分配内存，加上硬驱逐阈值的总和，那么Linux内核的OOM killer将介入并强行终止Pod容器中的1个或N个进程。 ([View Highlight](https://read.readwise.io/read/01j3dcghk35x82x1swjbmaeet6))