title:: Go垃圾回收调优 (highlights)
author:: [[MySpace]]
full-title:: "Go垃圾回收调优"
category:: #articles
url:: https://www.hitzhangjie.pro/blog/2022-11-10-go%E5%9E%83%E5%9C%BE%E5%9B%9E%E6%94%B6%E8%B0%83%E4%BC%98/
summary:: Go's garbage collection (GC) process can be tuned using the GOGC variable and the newer GOMEMLIMIT feature to manage memory usage effectively. The ballast approach previously used had issues, such as high physical memory consumption and potential out-of-memory (OOM) errors. The recommended strategy now is to set GOMEMLIMIT for soft memory limits while dynamically adjusting GOGC for optimal GC frequency and performance.
![](https://www.hitzhangjie.pro/doks.png)

- Highlights first synced by [[Readwise]] [[Dec 17th, 2024]]
	- 我们应该根据实际部署情况来界定各个进程允许的资源使用上限来确定GOMEMLIMIT的值：
	  
	  •   比如没有混部，per container per service或per host per service，那么可以用70%的内存资源作为软限制值，这样GC频率控制比较好，又留了较多的buffer给系统中其他服务，这也是uber容器化部署的一个经验值。
	  •   再比如有混布，一台机器部署了10个服务，那每个进程允许的软限制值肯定不能继续用机器内存的70%这个值，应该划分的更小，比如平均下7%，或者针对不同进程的实际情况在服务配置文件中进行指定。 ([View Highlight](https://read.readwise.io/read/01jenz8vw8y6zsg3yjq0x8wgqb))