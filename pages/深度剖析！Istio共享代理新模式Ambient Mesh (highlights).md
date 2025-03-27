title:: 深度剖析！Istio共享代理新模式Ambient Mesh (highlights)
author:: [[华为云云原生团队]]
full-title:: "深度剖析！Istio共享代理新模式Ambient Mesh"
category:: #articles
url:: https://mp.weixin.qq.com/s/B0q73ACAvmY4SjW42A2GVw
summary:: Istio社区Ambient Mesh开源，Sidecar模式 vs Ambient模式。
![](https://mmbiz.qpic.cn/mmbiz_jpg/ia1Z7HH4plnAlstIxcrhX8sAlGAx2AAceIiaGUmwVJXxHqw7E1bIAmtvqynUp6V6XocIEjzC0UzmB3L14cXYb68g/0?wx_fmt=jpeg)

- Highlights first synced by [[Readwise]] [[Apr 10th, 2024]]
	- 在 Sidecar 模式下，Istio 通过 InitContainer 或 Istio-CNI 实现流量拦截。Ambient Mesh下 Istio-CNI 是必选组件，下图展示了基本的 Ambient Mesh四层治理架构：
	  
	  ![图片](https://mmbiz.qpic.cn/mmbiz_png/ia1Z7HH4plnAlstIxcrhX8sAlGAx2AAcesuyI8fI4Gariao0To1UcWMiaqQWWejzibdJUNE64PHGiaCMXuHdxIjJm3g/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
	  
	  图3  Ambient Mesh四层治理架构
	  
	  Istio-CNI 中，新增 Ambient 的处理模块，该模块会监听Namespace 以及 Pod的变化，为所在节点的应用设置路由和iptables规则:
	  
	  •   **路由：**设置路由表，将本节点应用发出的流量路由到 ztunnel，以及将本节点接收的流量路由到ztunnel。
	    
	  •   **iptables：**在ztunnel容器中设置iptables规则，将流量透明拦截至 ztunnel 对应的端口上。
	    
	  
	  ztunnel 是 Ambient 新引入的组件，以 Daemonset 的方式部署在每个节点上。ztunnel 为网格中的应用通信提供 mTLS、遥测、身份验证和 L4授权功能，不执行任何七层协议相关的处理。只有当ztunnel运行在工作负载相同的节点上时，控制面才会将工作负载证书颁发给该 ztunnel。因此 当ztunnel 被攻击时，只有运行在该节点上的负载的证书可能被盗用，安全风险相对可控，这和其他实现良好的节点共享基础设施类似。 ([View Highlight](https://read.readwise.io/read/01hv1z0xfg2ssh3b9bbasp7qww))
	- sleep访问productpage的流量被同节点的tunnel以TPROXY（透明代理）方式拦截转发到ztunnel(监听127.0.0.1:15001)，使用TPROXY的好处是保留原始的目的地址，ztunnel做转发时必须依赖原始目的地址。 ([View Highlight](https://read.readwise.io/read/01hv1z40185bpkh6njcj4dcmzz))
		- 💡: 关键点：使用tproxy可以保留原始的目的地址
	- **Ambient Mesh四层流量治理小结**
	  id:: 67d7a09a-8e3c-4db8-9593-d4ba89814f3c
	  
	  ![图片](https://mmbiz.qpic.cn/mmbiz_png/ia1Z7HH4plnAlstIxcrhX8sAlGAx2AAceCgxO2cKbgVcZmA227Ids05NibEmBtlwiauWltP1fhzKeibY1tXbZrXX2g/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
	  
	  图14 完整的服务访问四层代理
	  
	  sleep访问productpage的实例中，虽然我们使用的是HTTP协议，但是从Ambient所有的组件视角来看，其代理的为TCP流量。前面我们深入分析了ztunnel中每一个监听器、每一个Cluster的工作原理，看起来可能会很复杂。故在此通过图14进行一个概要的总结，我们发现在通信的过程中，实际参加工作的模块并不多：
	  
	  1.  发送侧的路由、iptables：将流量拦截到ztunnel的15001端口
	    
	  2.  发送侧ztunnel：两个监听器和对应的两个cluster
	    
	  3.  接收侧的路由、iptables：将流量拦截到ztunnel的15008端口
	    
	  4.  接收ztunnel：virtual_inbound监听器及关联的cluster ([View Highlight](https://read.readwise.io/read/01hv1z6zzfe97x1a31p76drkfe)) #card