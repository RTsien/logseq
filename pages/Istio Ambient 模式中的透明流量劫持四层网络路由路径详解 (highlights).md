title:: Istio Ambient 模式中的透明流量劫持四层网络路由路径详解 (highlights)
author:: [[Jimmy Song]]
full-title:: "Istio Ambient 模式中的透明流量劫持四层网络路由路径详解"
category:: #articles
url:: https://jimmysong.io/blog/ambient-mesh-l4-traffic-path/
summary:: 本文以图示和实际操作的形式详细介绍了 Ambient Mesh 中的透明流量劫持和四层（L4）流量路径。
![](https://jimmysong.io/images/banner/ambient-l4.jpg)

- Highlights first synced by [[Readwise]] [[Apr 10th, 2024]]
	- 什么是 HBONE？[src](https://jimmysong.io/blog/ambient-mesh-l4-traffic-path#what-is-hbone)
		- HBONE 是 HTTP-Based Overlay Network Environment 的缩写，是一种使用 HTTP 协议提供隧道能力的方法。客户端向 HTTP 代理服务器发送 HTTP CONNECT 请求（其中包含了目的地址）以建立隧道，代理服务器代表客户端与目的地建立 TCP 连接，然后客户端就可以通过代理服务器透明的传输 TCP 数据流到目的服务器。在 Ambient 模式中，Ztunnel（其中的 Envoy）实际上是充当了透明代理，它使用 [Envoy Internal Listener](https://www.envoyproxy.io/docs/envoy/latest/configuration/other_features/internal_listener) 来接收 HTTP CONNECT 请求和传递 TCP 流给上游集群。 ([View Highlight](https://read.readwise.io/read/01hv1narreajem7beth91q9v9n))
- New highlights added [[Apr 20th, 2024]] at 2:34 PM
	- Outbound 流量劫持[](https://jimmysong.io/blog/ambient-mesh-l4-traffic-path#outbound)
	  
	  Ambient mesh 的 pod 出站流量的透明流量劫持流程如下：
	  
	  1.  Istio CNI 在节点上创建 `istioout` 网卡和 iptables 规则，将 Ambient mesh 中的 Pod IP 加入 [IP 集](https://ipset.netfilter.org/) ，并通过 netfilter `nfmark` 标记和路由规则，将 Ambient mesh 中的出站流量通过 Geneve 隧道透明劫持到 `pistioout` 虚拟网卡；
	  2.  ztunnel 中的 init 容器创建 iptables 规则，将 `pistioout` 网卡中的所有流量转发到 ztunnel 中的 Envoy 代理的 15001 端口；
	  3.  Envoy 对数据包进行处理，并与上游端点建立 HBONE 隧道（HTTP CONNECT），将数据包转发到上游。 ([View Highlight](https://read.readwise.io/read/01hvx0t1bqbt1ajqgsdn916dt4))