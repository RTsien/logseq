title:: 使用 ztunnel 实现 Layer 4 网络和 mTLS (highlights)
author:: [[Istio]]
full-title:: "使用 ztunnel 实现 Layer 4 网络和 mTLS"
category:: #articles
url:: https://istio.io/latest/zh/docs/ops/ambient/usage/ztunnel/
summary:: The text explains how to use ztunnel for Layer 4 networking and mTLS in Istio. It provides instructions for checking ztunnel status, verifying ztunnel traffic logs, and validating ztunnel load balancing. The text also covers monitoring through Prometheus, Grafana, and Kiali, and addresses L4 authentication policies and interoperability between Ambient and non-Ambient endpoints.
![](https://readwise-assets.s3.amazonaws.com/media/uploaded_book_covers/profile_1160846/istio-social.png)

- Highlights first synced by [[Readwise]] [[Apr 10th, 2024]]
	- ztunnel（Zero Trust Tunnel，零信任隧道）组件是专门为 Istio Ambient 网格构建的基于每个节点的代理。由于工作负载 Pod 不再需要在 Sidecar 中运行代理也可以参与网格，因此 Ambient 模式下的 Istio 也被非正式地称为 “无 Sidecar” 网格。 ([View Highlight](https://read.readwise.io/read/01hv1hg50ee7rsabnnkg91q5w9))
	- HBONE（HTTP Based Overlay Network Encapsulation，基于 HTTP 的覆盖网络封装）是 Istio 中特定的术语。 它是指通过 [HTTP CONNECT](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/CONNECT)9 方法使用标准 HTTP 隧道来透明地传递应用程序数据包/字节流。 在 Istio 的当前实现中，它通过使用 HTTP CONNECT 方法透明地隧道传输 TCP 数据包， 使用 [HTTP/2](https://httpwg.org/specs/rfc7540.html)10， 并通过[双向 TLS](https://www.cloudflare.com/learning/access-management/what-is-mutual-tls/)11 提供加密和相互身份验证且 HBONE 隧道本身在 TCP 端口 15008 上运行。 来自 IP 层的整体 HBONE 数据包格式如下图所示。
	  
	  [![](https://istio.io/latest/zh/docs/ops/ambient/usage/ztunnel/hbone-packet.png)](https://istio.io/latest/zh/docs/ops/ambient/usage/ztunnel/hbone-packet.png)
	  
	  HBONE L3 数据包格式 ([View Highlight](https://read.readwise.io/read/01hv1p210gmbc86k9tbpaawtg3)) #[[card]]
	- 如果目标是具有多个端点的服务，ztunnel 代理会自动执行客户端负载均衡。 无需额外配置。ztunnel 负载均衡算法是内部固定的 L4 循环算法， 根据 L4 连接状态分配流量，用户不可配置。 ([View Highlight](https://read.readwise.io/read/01hv1pdbdbr9v39f428knp3rm2))
	- 这是一种循环负载均衡算法，并且独立于可以在 `VirtualService` 的 `TrafficPolicy` 字段中配置的任何负载均衡算法，因为如前所述，`VirtualService` API 对象的所有方面都被实例化 在 Waypoint 代理上而不是 ztunnel 代理上。 ([View Highlight](https://read.readwise.io/read/01hv1pd1dgy8mthgyqm025437t))
	- Ambient 模式和 Sidecar 模式的 Pod 选择逻辑
	  
	  具有 Sidecar 代理的 Istio 可以与同一计算集群中基于 Ambient 的节点级代理共存。 确保相同的 Pod 或命名空间不会配置为同时使用 Sidecar 代理和 Ambient 节点级代理非常重要。 但是，如果确实发生这种情况，当前此类 Pod 或命名空间将优先进行 Sidecar 注入。
	  
	  请注意，理论上，可以通过将各个 Pod 与命名空间标签分开标记来将同一命名空间中的两个 Pod 设置为使用不同的模式，但不建议这样做。对于大多数常见用例， 建议对单个命名空间内的所有 Pod 使用单一模式。
	  
	  确定 Pod 是否设置为使用 Ambient 模式的确切逻辑如下。
	  
	  1.  在 `cni.values.excludeNamespaces` 配置中的 `istio-cni` 插件配置排除列表用于跳过排除列表中的命名空间。
	  2.  Pod 已使用 `ambient` 模式，如果：
	  
	  •   命名空间具有 `istio.io/dataplane-mode=ambient` 标签
	  •   Pod 上不存在 `sidecar.istio.io/status` 注解
	  •   `ambient.istio.io/redirection` 不是 `disabled`
	  
	  避免配置冲突的最简单选项是用户确保对于每个命名空间， 它要么具有 Sidecar 注入标签（`istio-injection=enabled`）， 要么具有 Ambient 数据平面模式标签（`istio.io/dataplane- mode=ambient`）， 但绝不能两者兼而有之。 ([View Highlight](https://read.readwise.io/read/01hv1pf4qp2vn0rr2a4r8nwcg3))