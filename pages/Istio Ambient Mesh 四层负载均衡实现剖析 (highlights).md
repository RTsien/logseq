title:: Istio Ambient Mesh 四层负载均衡实现剖析 (highlights)
author:: [[aliyun.com]]
full-title:: "Istio Ambient Mesh 四层负载均衡实现剖析"
category:: #articles
url:: https://developer.aliyun.com/article/1290950
summary:: This text discusses how Istio Ambient Mesh implements layer 4 load balancing by bypassing Kubernetes' load balancing rules using iptables. It explains the process of intercepting traffic and encrypting it before forwarding it to the destination workload through the ztunnel component. The text also delves into the analysis of iptables rules, network packet capturing, and code analysis to explain how Istio Ambient Mesh achieves layer 4 load balancing.
![](https://img.alicdn.com/tfs/TB1LCE1aQ5E3KVjSZFCXXbuzXXa-200-200.png)

- Highlights first synced by [[Readwise]] [[Apr 10th, 2024]]
	- [[Istio Ambient]]四层负载均衡 #card
	  id:: 6617779c-82dd-41bf-be3a-60fd211d5075
		- 由于Istio Ambient Mesh提供了四层流量安全的能力，Istio Ambient Mesh的设计要求流量从源Pod离开后，首先进入ztunnel Pod，在ztunnel内完成负载均衡、加密（如果启用）后，发往对端Pod所在的节点，再进入ztunnel Pod，并由ztunnel pod将流量解密，再透传至目标Pod。
		  
		  跳过K8s service iptables规则
		  
		  我们先看一下宿主机上的iptables规则，来了解Istio如何跳过了K8s的负载均衡
		  
		    mangle*
		   ` -A PREROUTING -j ztunnel-PREROUTING #  Istio加入的ztunnel-PREROUTING链`
		  
		  可以看到，Istio在PREROUTING链中插入了一条规则，当流量命中这条规则时，将跳转到ztunnel-PREROUTING链继续执行。在这个链中，将为数据包打上0x100标记：
		  
		    *mangle
		    ......
		  `  -A ztunnel-PREROUTING -p tcp -m set --match-set ztunnel-pods-ips src -j MARK --set-xmark 0x100/0x100`
		  
		  mangle表的PREROUTING链处理完毕后，进入nat表的PREROUTING链，Istio又在K8s的service规则前插入了一条规则，直接跳转到ztunnel-PREROUTING.
		  
		    *nat
		    -A PREROUTING -j ztunnel-PREROUTING
		    -A PREROUTING -m comment --comment "kubernetes service portals" -j KUBE-SERVICES
		    -A PREROUTING -d 172.18.0.1/32 -j DOCKER_OUTPUT
		  
		  而nat表的ztunnel-PREROUTING链则直接接受了带有0x100标记的数据包，从而使得其跳过了后续k8s service的iptables规则:
		  
		    *nat
		    -A ztunnel-PREROUTING -m mark --mark 0x100/0x100 -j ACCEPT
		  
		  0x100标记除了在iptables规则中起到了跳过K8s service规则的作用以外，还被用于策略路由，以使得流量被重定向到ztunnel，但本文主要聚焦负载均衡，故这一部分不展开讨论。 ([View Highlight](https://read.readwise.io/read/01hv1ky15re1qp28x088dm5jjw))
	- ztunnel pod内抓包验证
	  
	  我们通过sleep应用内的curl访问httpbin服务，并在sleep应用pod所在的宿主机上运行的ztunnel pod内抓包，以验证上述规则生效，应用和服务的部署信息为：
	  
	  类型
	  
	  名称
	  
	  地址:端口
	  
	  Pod
	  
	  sleep-bc9998558-8sf7z
	  
	  10.244.2.8:Random
	  
	  Service
	  
	  httpbin.default.svc.cluster.local
	  
	  10.96.130.105.8000
	  
	  通过tcpdump -i any 'src host 10.244.2.8'指令进行抓包：
	  
	    03:26:11.610298 pistioout In  IP 10.244.2.8.48920 > 10.96.130.105.8000: Flags [S], seq 294867080, win 64240, options [mss 1460,sackOK,TS val 1553210180 ecr 0,nop,wscale 7], length 0
	    03:26:11.610336 eth0  In  IP 10.244.2.8.48920 > 10.96.130.105.8000: Flags [.], ack 286067347, win 502, options [nop,nop,TS val 1553210181 ecr 61738018], length 0
	    03:26:11.610377 eth0  In  IP 10.244.2.8.48920 > 10.96.130.105.8000: Flags [P.], seq 0:75, ack 1, win 502, options [nop,nop,TS val 1553210181 ecr 61738018], length 75
	  
	  通过在ztunnel pod内抓包，可以验证进入ztunnel的数据包仍然保持了服务地址（ClusterIP）10.96.130.105.8000。
	  
	  您可能注意到，为什么第一个数据包来自pistioout而后续的数据包来自eth0？我将在后续的文章中对这部分进行分享。 ([View Highlight](https://read.readwise.io/read/01hv1kzsn06k4a8wf1n80hx4ej))