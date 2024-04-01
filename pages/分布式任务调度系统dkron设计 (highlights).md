title:: 分布式任务调度系统dkron设计 (highlights)
author:: [[woa.com]]
full-title:: "分布式任务调度系统dkron设计"
category:: #articles
url:: https://iwiki.woa.com/p/377034244
![](https://readwise-assets.s3.amazonaws.com/static/images/article2.74d541386bbf.png)

- Highlights first synced by [[Readwise]] [[Mar 29th, 2024]]
	- dkron在启动的时候会扫描所有以“-executor”和“-processor”结尾的二进制文件，以子进程的方式拉起这些插件，和插件之间通过本地domain socket进行grpc通信。 ([View Highlight](https://read.readwise.io/read/01ht4hdz2fdf6nqkfqy3fwe004))