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