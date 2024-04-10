title:: KubeCon EU 2024 巴黎见闻与回顾 (highlights)
author:: [[Jimmy Song - 专注于探索后 Kubernetes 时代的云原生新范式 – 博客]]
full-title:: "KubeCon EU 2024 巴黎见闻与回顾"
category:: #articles
url:: https://jimmysong.io/blog/kubecon-eu-paris-recap/
summary:: 上周我在巴黎参加了 KubeCon EU 2024
，这也是我第一次参加中国以外的 KubeCon。本次大会可谓盛况空前，据说有 1.2 万人参加了会议。本文将为你分享我对本次 KubeCon 的一些观察，主要着重在我关注的服务网格与云原生基础架构领域。


Istio Contributor 在 KubeCon EU Istio 展台


Istio、Cilium 及服务网格
Istio
 和 Service Mesh 成为了热门讨论的话题，集中展示了在云原生生态系统中这两项技术的最新进展和应用。本次大会涵盖了从基础设施优化、数据本地化、分布式追踪到多集群部署等多个领域，反映了 Service Mesh 技术在实际应用中的广泛关注和持续创新。
数据本地化和全局请求路由
Pigment 的 Arthur Busser 和 Baudouin Herlicq 分享了如何利用 Kubernetes 和 Istio 实现数据本地化的需求。他们介绍了利用 Istio 基于自定义头部进行请求路由的方法，这对于满足如 GDPR 和 CCPA 等法规的数据驻留要求至关重要。
分布式跟踪和可观测性增强
ThousandEyes (part of Cisco) 的 Chris Detsicas 探讨了如何配置 Istio 以使用 OpenTelemetry 实现有效的分布式跟踪，这为微服务生态系统提供了宝贵的可见性，有助于问题诊断和性能优化。
多集群部署和流量管理
China Mobile 的 Haiwen Zhang 和 Yongxi Zhang 介绍了一个简化 Istio 多集群部署的新方法，该方法使用一个全局唯一的 Istio 控制平面，通过主集群的 Apiserver 实现全局服务发现，自动连接多个集群的容器网络，为 Pod 提供直接网络连接。特别强调了 Kosmos 项目
，它提供了一种新的解决方案，以简化多集群环境下的服务网格部署和管理。
Google 的 Ameer Abbas 和 John Howard 探讨了如何在基础设施可靠性为 99.9% 的情况下构建出 99.99% 可靠性的服务，并提出了一系列应用架构原型（Archetypes），这些原型可以帮助设计和实现高可靠性的多集群应用程序。

原型 1：活动 - 被动区域（Active Passive Zones） - 在单个区域的两个区域部署所有服务，使用 SQL 数据...
![](https://jimmysong.io/blog/images/favicon-144.png)

- Highlights first synced by [[Readwise]] [[Apr 1st, 2024]]
	- Benjamin Leggett 和 Yuval Kohavi 引入了一种创新的方法，使 Istio 的 Amibent mode 能够支持任意 Kubernetes CNI，详见 [Istio 博客](https://istio.io/latest/zh/blog/2024/inpod-traffic-redirection-ambient/) 。这一进步解决了 Ambient mesh 中 CNI 支持有限的问题，无需重启应用程序 Pod 即可将其纳入 Ambient mode，这对于简化操作和降低基础设施成本具有重要意义。 ([View Highlight](https://read.readwise.io/read/01htcjeed93y7rqc29kv5pwpcq))