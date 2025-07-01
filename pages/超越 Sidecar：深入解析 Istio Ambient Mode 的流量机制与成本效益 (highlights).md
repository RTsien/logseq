title:: 超越 Sidecar：深入解析 Istio Ambient Mode 的流量机制与成本效益 (highlights)
author:: [[Jimmy Song]]
full-title:: "超越 Sidecar：深入解析 Istio Ambient Mode 的流量机制与成本效益"
category:: #articles
url:: https://jimmysong.io/blog/beyond-sidecar/
summary:: 欢迎阅读我的这篇博客——“超越 Sidecar：深入解析 Istio Ambient Mode 的流量机制与成本效益”。本文内容源自我在 KCD 北京的一次演讲。主要探讨的是 Istio 全新推出的一种数据面模式 —— Ambient Mode。它的核心理念是去除 Sidecar，减少资源开销与运维复杂度。本文将带大家了解 Ambient Mode 的出现背景、核心组件、流量路径机制以及与现有 Sidecar 模式的对比，从而帮助你快速评估并上手这项新特性。
点击查看幻灯片。
为什么关注 Ambient Mode？
首先，我们来思考一个问题：为什么要关注、甚至尝试这种新模式？Sidecar 在服务网格里一直都用得好好的，为什么要“去 Sidecar”呢？
让我们看看当前服务网格面临的一些问题和挑战。
服务网格的挑战

Sidecar 代理带来的 资源开销 和 运维复杂度
升级 或 重启 Envoy 时，常常需要连带重启所有 Pod
越来越多对 高性能、低成本 的需求


思考：有没有一种方式在保留服务网格核心能力（安全、可观测、流量控制）的同时，减少对每个 Pod 的侵入和额外资源消耗？

服务网格的几种部署模式


代理的位置

服务网格架构一直在探索代理部署位置的多种可能性。例如：

Sidecar：每个 Pod 内跑一个 Envoy。
Ambient：将代理从 Pod 中剥离到节点级（即本篇要谈的模式）。
Cilium Mesh：利用 eBPF 在内核空间做 L4，然后结合 Envoy 提供 L7 功能。
gRPC：直接将网格能力集成到 SDK 中。

这些模式在功能、安全、性能和管理复杂度上都有不同的侧重。Istio Ambient Mode 则是针对 Sidecar 带来的高资源消耗和运维成本，而提出的新尝试。
Ambient Mode 的诞生

Istio 的新一代架构，移除 Sidecar，通过 ztunnel + Waypoint Proxy 实现数据面的轻量化。
节省资源、降低运维复杂度。
依然支持 mTLS、策略管控，并为需要 L7 功能的流量提供可选的 Waypoint Proxy。



部署模式象限

以下表格是对比常见服务网格部署模式的一些简要特点：



模式
安全性
效率
可管理性
性能




Sidecar 模式
高安全性，隔离的代理
资源使用率高
集中化管理但较为复杂
增加一定延迟


Ambient 模式
通过 ztunnel 提供安全性，仍在发展中
更高效，共享代理
管理更简单但功能在完善中
良好；跨可用区时需注意网络开销


Cilium mesh
中等安全性，基于 eBPF
内核级效率
配置复杂
可视场景不同而异


gRPC
应用集成安全，依赖应用自身
高效
更新管理复杂
低延迟，适用于实时场景



Istio Ambient Mode 核心概念
接下来我们正式进入第二部分，深入看看 Ambient Mode 的具体组件，包括 ztunnel、Waypoint Proxy 以及 Istio CNI 在其中扮演的角色。
Ambient Mode 的核心组件

ztunnel (L4)

以 Node 级代理的方式运行
负责 透明流量拦截、mTLS 加密
适用于大部分只需 L4 转发的流量


Waypoint Proxy (L7)

可选部署（根据命名空间 / Service / Pod 粒度灵活配置）
处理 HTTP / gRPC 等高级功能（鉴权、路由、可观测等）


Istio CNI

取代 istio-init 容器，负责流量劫持
兼容 Sidecar 模式和 Ambient 模式
允许在非特权模式下为 Pod 设置流量重定向



Ambient 模式整体架构


Istio Ambient 模式架构

在 Ambient 模式下，Istio 数据面可分为两层：

安全层 (ztunnel)：每个节点部署一个轻量级 L4 代理。
可选的 L7 层 (Waypoint Proxy)：需要 HTTP/gRPC 代理时才部署。

Control Plane 依然由 Istiod 提供，主要负责给 ztunnel、Waypoint 下发配置和证书。
Waypoint Proxy 部署策略

Namespace 级（默认）：适用于该命名空间下所有 Workload
Service 级：仅特定关键服务需要 L7
Pod 级：更精细化控制
跨 Namespace：可以使用 Gateway 资源共享

Istio CNI

流量拦截：取代 istio-init 容器，使安装更加清晰简洁。
支持两种模式：兼容 Sidecar 模式 和 Ambient 模式。
非特权模式兼容性：允...
![](https://jimmysong.io/images/favicon.png)

- Highlights first synced by [[Readwise]] [[Mar 25th, 2025]]
	- 服务网格的几种部署模式
	  
	  ![](https://jimmysong.io/blog/beyond-sidecar/proxy-location.svg)
	  
	  代理的位置
	  
	  服务网格架构一直在探索代理部署位置的多种可能性。例如：
	  
	  •   **Sidecar**：每个 Pod 内跑一个 Envoy。
	  •   **Ambient**：将代理从 Pod 中剥离到节点级（即本篇要谈的模式）。
	  •   **Cilium Mesh**：利用 eBPF 在内核空间做 L4，然后结合 Envoy 提供 L7 功能。
	  •   **gRPC**：直接将网格能力集成到 SDK 中。 ([View Highlight](https://read.readwise.io/read/01jq2afbe4361prgdbwf62s35x))
	- Ambient Mode 的诞生
	  
	  •   Istio 的新一代架构，**移除 Sidecar**，通过 **ztunnel + Waypoint Proxy** 实现数据面的轻量化。
	  •   节省资源、降低运维复杂度。
	  •   依然支持 **mTLS、策略管控**，并为需要 L7 功能的流量提供可选的 **Waypoint Proxy**。
	  
	  ![](https://jimmysong.io/blog/beyond-sidecar/istio-data-plane-deployment-modes.svg) ([View Highlight](https://read.readwise.io/read/01jq2ahxxtxkwk2taa88dsfv3x))
		- 💡: mtls是什么？