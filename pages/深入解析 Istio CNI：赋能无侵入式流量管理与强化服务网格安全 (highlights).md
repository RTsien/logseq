title:: 深入解析 Istio CNI：赋能无侵入式流量管理与强化服务网格安全 (highlights)
author:: [[Jimmy Song - 专注于探索后 Kubernetes 时代的云原生新范式 – 博客]]
full-title:: "深入解析 Istio CNI：赋能无侵入式流量管理与强化服务网格安全"
category:: #articles
url:: https://jimmysong.io/blog/istio-cni-deep-dive/
summary:: 本文将深入探讨 Istio CNI 插件的设计理念、实现方式以及如何通过 Ambient Mode 来解决传统模式中存在的安全和权限问题。本文内容包括：

Init 容器的安全风险及其解决方案。
Istio CNI 的工作原理及其优势。
Ambient Mode 的实现机制及其与 CNI 的集成。

Istio 网络要求与解决方案概览
Istio 服务网格通过 Sidecar 模式实现应用流量的拦截和管理。该模式通过在应用程序 Pod 中注入 Sidecar Proxy 和 init 容器，并使用 iptables 规则来管理网络流量。详细的部署和操作过程请参见 Istio 中的 Sidecar 注入、透明流量劫持及流量路由过程详解
。虽然此方法在多数 Kubernetes 平台上有效，但对高权限的依赖在多租户环境中引发了安全方面的担忧。
Istio-init 的局限性
Istio 在其网络配置初期采用了 istio-init 容器来初始化流量拦截规则，这需要容器具有高级权限来修改网络配置，如 IPTables 规则。虽然这种方法实现了对流量的有效管理，但它也显著提高了权限需求并增加了安全风险。根据 Istio 官方文档
，istio-init 容器默认被注入到 Istio 网格中的 Pod 里，以便将网络流量劫持到 Istio 的 Sidecar 代理。这一过程需要对部署 Pod 的 Service Account 赋予 NET_ADMIN 容器权限
，可能与某些组织的安全政策相悖。
Istio CNI 插件
为响应这一挑战，Istio 社区推出了 Istio CNI
 插件，该插件避免了对 init 容器的需求，允许直接在 Kubernetes 的网络层面操作，从而降低权限需求并简化部署流程，但是存在 CNI 兼容性问题。
Ambient 模式的引入
Istio 的 Ambient Mode 是一种创新的无 sidecar 方案，它通过 使用 Geneve 隧道
 或 Istio CNI 提高网络的灵活性和安全性。
直到最近 Istio 社区才推出适配任意 CNI
 的通用的解决方案。此模式解决了与任意 CNI 的兼容性问题，使 Istio 能够在不影响现有网络策略的前提下，更有效地管理服务间的流量。
NET_ADMIN 权限的安全考虑
在 Kubernetes 和 Docker 等容器化环境中，NET_ADMIN...
![](https://jimmysong.io/blog/images/favicon-144.png)

- Highlights first synced by [[Readwise]] [[Apr 20th, 2024]]
	- Istio CNI 插件是一个二进制文件，作为代理安装在每个节点的文件系统中。以下流程图说明了 Istio CNI 节点代理的工作原理：
	  
	  •   Istio CNI Node Agent 充当安装在每个节点上的代理。
	  •   安装 Istio CNI 插件并更新节点的 CNI 配置。
	  •   代理监控 CNI 插件和配置路径的更改。
	  •   在 Sidecar 模式下，它使用 pod 的 iptables 处理 sidecar 网络设置。
	  •   在 Ambient 模式下，它将 pod 事件同步到环境监控服务器，然后该服务器在 pod 内配置 iptables。
	  •   节点需要提升权限，例如 `CAP_SYS_ADMIN` 、 `CAP_NET_ADMIN` 和 `CAP_NET_RAW` 才能在任一模式下运行。 ([View Highlight](https://read.readwise.io/read/01hvx29vxhymk0wxbemvpa086z))
	- Istio 的 Ambient Mode 是为了适配所有 CNI 而设计的一种模式，它通过 ztunnel 来透明地处理 Pod 内的流量转发，而不影响现有的 CNI 配置。这种模式下，Ambient Mode 通过 ztunnel 管理流量，使其流经 Istio 服务网格，而标准的 CNI 则侧重于为 Pod 提供标准化的网络接入。
	  
	  CNI 的主要职责是解决 Kubernetes Pod 之间的网络连通性，例如分配 IP 地址和转发数据包。相比之下，Ambient Mode 需要将流量导入 ztunnel，这与 CNI 的网络配置可能存在不兼容，主要问题包括：
	  
	  •   主流 CNI 的网络配置可能会与 Istio 的 CNI 扩展冲突，导致流量处理中断。
	  •   如果部署的网络策略依赖于 CNI，那么使用 Istio CNI 时可能会影响这些策略的执行。
	  
	  为解决这些问题，可以通过将 ztunnel 运行在与 Pod 相同的用户空间中，避免与 CNI 修改过的内核空间的冲突。这样，Pod 可以直接连接到 ztunnel，绕过 CNI 的影响。 ([View Highlight](https://read.readwise.io/read/01hvx05gbh05wea4fgmwj07ayw))
	- 解决 Istio Ambient Mode 和 Kubernetes CNI 的冲突
	  
	  为缓解这些冲突，Istio 的 Ambient Mode 避免了对 CNI 修改过的内核空间的依赖：
	  
	  •   **在用户空间运行 ztunnel**：这一策略让 ztunnel 与 Pod 运行在同一用户空间，避免了与 CNI 的直接冲突。
	  •   **确保 CNI 兼容性**：Istio CNI 配置必须在不影响现有 CNI 插件配置的前提下进行，确保 Pod 间的正常通信和流量管理。
	  
	  这些措施帮助 Istio 的 Ambient Mode 在不干扰现有 CNI 插件的情况下，有效管理服务间流量。 ([View Highlight](https://read.readwise.io/read/01hvx0anmjqvr7rvf5x397w7n7))
	- Istio CNI 插件的设计理念、实现方式和优势，特别是Istio CNI 如何解决了传统 `istio-init` 方法中存在的权限和安全问题。通过这些创新，Istio 在网络安全和操作简便性上取得了重大进步，为 Kubernetes 环境中实施 Istio 提供了更灵活和高效的方法。 ([View Highlight](https://read.readwise.io/read/01hvx2dsc9k1yzzspz2vp0m38p))