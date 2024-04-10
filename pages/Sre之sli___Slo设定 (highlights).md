title:: Sre之sli/Slo设定 (highlights)
author:: [[运维开发故事]]
full-title:: "Sre之sli/Slo设定"
category:: #articles
url:: https://zhuanlan.zhihu.com/p/458324494
summary:: SLI/SLO are indicators and objectives used to measure system stability. Choosing the right SLI metrics is crucial for monitoring system performance. VALET method helps in selecting SLI metrics based on Volume, Availability, Latency, Error, and Ticket dimensions.
![](https://picx.zhimg.com/v2-84528acb02937ded1f642f6d26835446_720w.jpg?source=172ae18b)

- Highlights first synced by [[Readwise]] [[Apr 10th, 2024]]
	- SLI，全名Service Level Indicator，是服务等级指标的简称，它是衡定系统稳定性的指标。
	- SLO，全名Sevice Level Objective，是服务等级目标的简称，也就是我们设定的稳定性目标。
	  
	  简单一句话：SLI 就是我们要监控的指标，SLO 就是这个指标对应的目标。 ([View Highlight](https://read.readwise.io/read/01hv1we5hfkbayzk5t7ery45nk))
	- 如何选择SLI
	  
	  在系统中，常见的指标有很多种，比如：
	  
	  •   系统层面：CPU使用率、内存使用率、磁盘使用率等  
	    
	  •   应用服务器层面：端口存活状态、JVM的状态等  
	    
	  •   应用运行层面：状态码、时延、QPS、TPS以及连接数等  
	    
	  •   PASS层面：mysql、redis、kafka、mq和分布式文件储存等组件的QPS、TPS、时延等。  
	    
	  
	  这么多指标，应该如何选择呢？只要遵从两个原则就可以：
	  
	  •   选择能够标识一个主体是否稳定的指标，如果不是这个主体本身的指标，或者不能标识主体稳定性的，就要排除在外。  
	    
	  •   优先选择与用户体验强相关或用户可以明显感知的指标。  
	    
	  
	  我们可以直接套用 Google 的方法：VALET。VALET 是 5 个单词的首字母，分别是 Volume、Availability、Latency、Error 和 Ticket。这 5 个单词就是我们选择 SLI 指标的 5 个维度。 ([View Highlight](https://read.readwise.io/read/01hv1wfacf7k9b0tvvcphkb65t))