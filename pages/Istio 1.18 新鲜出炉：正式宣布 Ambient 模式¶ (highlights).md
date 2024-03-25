title:: Istio 1.18 新鲜出炉：正式宣布 Ambient 模式¶ (highlights)
author:: [[DaoCloud]]
full-title:: "Istio 1.18 新鲜出炉：正式宣布 Ambient 模式¶"
category:: #articles
url:: https://docs.daocloud.io/blogs/230609-istio118.html#_4
summary:: Istio在1.18版本中引入了Ambient模式，旨在简化操作、提高资源利用率，并改善安全性。这种模式不需要在应用程序Pod中注入边车代理，提供更高的性能和更简单的安全配置。除了Ambient模式，该版本还带来了对Kubernetes Gateway API的多项重要改进和修改，以及istioctl的增强功能。新的并发配置逻辑旨在在部署类型之间保持一致，提高性能和资源利用。虽然Ambient模式目前处于Alpha阶段，但为用户提供了更多灵活性，未来版本中预计会有更多更新和改进。
![](https://readwise-assets.s3.amazonaws.com/static/images/article4.6bc1851654a0.png)

- Highlights first synced by [[Readwise]] [[Mar 24th, 2024]]
	- 侵入性强：在 Sidecar 模式中，每个应用程序 Pod 都需要注入一个 Envoy 代理。 这就导致一些应用程序必须在设计和编码阶段考虑到与 Sidecar 代理的交互。此外， 当网格升级时，Sidecar 的升级过程可能会影响应用程序，不能很好地实现网格能力和业务容器的解耦，从而增加了应用开发和运维的复杂性。 ([View Highlight](https://read.readwise.io/read/01hspgwaz2ep0pcnxfc6zr9hyp))
		- 💡: 具体要考虑哪些方面？
	- 性能：在早期，Istio 采用了基于 Envoy 的共享代理模式，然而在开发过程中，发现 Envoy 存在配置繁琐等问题， 因此 Istio 开发了自己的基于 Rust 版本的共享代理（ztunnel）。在未来，预计环境网格将与传统的边车模式具有相当的性能表现。 ([View Highlight](https://read.readwise.io/read/01hsph2hptqpr48nq8w1c2esyy))
		- 💡: 学习一下ztunnel的代码
	- 安全性：环境网格通过在每个节点上运行一个共享代理 Ztunnel 来确保安全性。这个代理被部署在每个节点上， 负责处理所有进出节点的流量，可以提供与传统 Istio 网格中边车代理相同的安全性。 一旦为命名空间启用 Ambient 模式，就会创建一个安全覆盖层。 这个覆盖层为应用程序提供 mTLS、遥测、身份验证和 L4 授权等功能，而无需终止或解析 HTTP 流量。 ([View Highlight](https://read.readwise.io/read/01hsph68amq5zes79537n3c7yx))
		- 💡: l4授权指什么？是在ztunnel里做的吗？mtls具体是什么？