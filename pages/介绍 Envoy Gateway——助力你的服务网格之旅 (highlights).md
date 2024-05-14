title:: 介绍 Envoy Gateway——助力你的服务网格之旅 (highlights)
author:: [[Jimmy Song - 专注于探索后 Kubernetes 时代的云原生新范式 – 博客]]
full-title:: "介绍 Envoy Gateway——助力你的服务网格之旅"
category:: #articles
url:: https://jimmysong.io/blog/envoy-gateway-introduction/
summary:: 在 Kubernetes 环境下选择正确的网络通信工具至关重要。根据Tetrate 的讨论
，选择取决于网络通信的类型：南北向流量还是东西向流量。对于主要处理外部请求的服务，Envoy Gateway 是理想选择，它不仅高效管理流量，还能在你向微服务架构过渡时提供无缝集成。
本文将探讨 Envoy Gateway 在 Kubernetes 上部署的优势，及其它与服务网格的关系，展示为何它是暴露服务到公网的理想选择。
Envoy Gateway 及其在服务网格中的角色概述
Envoy Gateway 是一个围绕 Envoy Proxy 构建的 Kubernetes 原生 API 网关，它旨在降低用户采用 Envoy 作为 API 网关的难度，并为供应商建立 API 网关（例如 Tetrate Enterprise Gateway for Envoy
）增值产品奠定基础。
Envoy Gateway 不仅是管理南北流量的理想选择，也可作为连接和保护服务网格中服务的关键组件。它还通过提供安全的数据传输、流量路由、负均衡及故障恢复等功能，增强了微服务之间的通信效率和安全性。Envoy Gateway 利用其内置的 Envoy Proxy 技术，可以处理大量的并发连接和复杂的流量管理策略，同时保持较低的延迟和高吞吐量。
此外，Envoy Gateway 与 Kubernetes Gateway API 的紧密集成使得它能够以声明式的方式进行配置和管理，极大简化了服务网格中网关的部署和更新过程。这种集成不仅提升了操作效率，还使得 Envoy Gateway 能够在不增加额外复杂性的前提下，与服务网格如 Istio 这样的解决方案无缝协作。
下图展示了 Envoy Gateway 与服务网格的关系。
graph TB
    subgraph Kubernetes["Kubernetes Cluster"]
        eg["Envoy Gateway"]
        svcs["Services"]
        pods["Pods"]
        gwapi["Kubernetes Gateway API"]
        eg -- "Manages North-South Traffic" --> svcs
        eg -- "Configured by" --> gwapi
        g...
![](https://jimmysong.io/blog/images/favicon-144.png)

- Highlights first synced by [[Readwise]] [[May 13th, 2024]]
	- Envoy Gateway 提供了几个核心功能，使其成为 API 网关的突出选择：
	  
	  •   **简化配置**：通过与 Kubernetes Gateway API 直接集成，Envoy Gateway 允许开发者使用 Kubernetes 自定义资源以声明方式配置路由规则、安全策略和流量管理。
	  •   **性能和可扩展性**：基于经过实战测试的 Envoy Proxy，它提供卓越的性能和可扩展性，轻松处理数千个服务和每秒数百万个请求。
	  •   **安全功能**：内置支持各种安全措施，如 SSL/TLS 终止、OAuth2、OIDC 认证以及细粒度访问控制。
	  •   **可观测性**：提供全面的监控能力，包括详细的度量、日志和追踪，这对于诊断和理解流量行为至关重要。 ([View Highlight](https://read.readwise.io/read/01hxnvn3hpgyr1jgr3np2xz9ty))
	- 在 Kubernetes 环境中引入的 Gateway API 为集成和配置 Ingress 网关提供了一种新的强大方法，它与传统的 Ingress 相比具有更高的灵活性和功能性。正如我在 [Gateway API：Kubernetes 和服务网格入口中网关的未来](https://jimmysong.io/blog/why-gateway-api-is-the-future-of-ingress-and-mesh/) 中所讨论的，Gateway API 通过区分角色和提供跨命名空间支持，更适应多云环境，且已被多数 API 网关采用。这种 API 设计支持了 ingress 网关（南北向流量）与服务网格（东西向流量，跨集群路由）的融合，使得 Envoy Gateway 成为 Kubernetes 和服务网格中统一未来的网关解决方案。通过引入 Gateway API，Envoy Gateway 强化了其作为云原生环境中前沿代理的角色，使得用户能够更灵活地管理其流量和策略。 ([View Highlight](https://read.readwise.io/read/01hxnvvagn5ct73dbe3aj64yqj))
	- Kubernetes Gateway API 是 Envoy Gateway 的基石，它提供了一种更具表达性、灵活性和以角色为导向的方式来配置 Kubernetes 生态系统中的网关和路由。该 API 提供了如 GatewayClass、Gateway、HTTPRoute 等自定义资源定义（CRD），Envoy Gateway 利用这些资源创建用户友好且一致的配置模型，与 Kubernetes 的原生原则保持一致 ([View Highlight](https://read.readwise.io/read/01hxnvwx80jrjk3gw4ywm1eshp))
	- 什么是 API Gateway？
	  
	  API Gateway 是对 API 的全面管理和托管服务。它作为应用程序与后端服务之间的中间层，不仅处理创建、维护、发布、运行和下线等生命周期事件，还承担着更多关键职能。一个完善的 API Gateway 应该提供以下功能来丰富和扩展其基本定义：
	  
	  1.  **流量控制**：API Gateway 应能够处理并控制到后端服务的流量，包括请求路由、负载均衡、熔断机制以及速率限制，以保证后端服务的稳定性和高可用性。
	  2.  **安全性保障**：应具备鉴权、授权和加密功能，能够有效地管理和保护 API 的安全。这涉及到身份验证机制、API 密钥管理、OAuth、JWT、mTLS 等，以确保只有授权的用户和服务能够访问 API。
	  3.  **监控和分析**：提供实时监控和日志记录功能，能够跟踪 API 的使用情况、性能指标、异常检测和分析流量模式，从而优化 API 的性能和响应能力。
	  4.  **变更管理**：支持对 API 变更进行管理，包括版本控制和渐进式部署（如蓝绿部署或金丝雀发布），以无缝过渡新版本且最小化对最终用户的影响。
	  5.  **请求和响应的转换**：允许对传入和传出的 API 调用进行转换，比如从 REST 到 GraphQL 的转换，或是添加、删除和修改请求头和响应头。
	  6.  **跨域资源共享（CORS）支持**：管理和控制跨域请求，允许不同域的前端应用安全地调用后端 API。
	  7.  **配额和计费**：为 API 使用设定配额限制，同时支持计费功能，以适用于商业化的 API 提供。
	  8.  **用户友好的开发者门户**：提供一个面向开发者的门户，使得第三方开发者可以轻松地发现、测试和集成 API。 ([View Highlight](https://read.readwise.io/read/01hxnx3411fe1b562y6nt4avdh))
	- •   **协议支持**：支持各种网络协议，包括 HTTP/HTTPS、WebSocket、gRPC 等，确保与多种客户端和服务的兼容性。
	  •   **插件化和扩展性**：允许通过插件或中间件来扩展 API Gateway 的功能，使其可以根据业务需求灵活适配各种中间件服务。
	  •   **服务治理**：集成服务注册和发现机制，以适应微服务架构下服务的动态性。 ([View Highlight](https://read.readwise.io/read/01hxnx3dsjhq10820hgwaq6a81))
	- 其他网关的比较
	  
	  与 Istio 的入口网关或 NGINX Ingress 等其他流行解决方案相比，Envoy Gateway 凭借其与 Kubernetes 的原生集成以及利用 Envoy 全部潜力的专注，而脱颖而出。下图从多方面对比了目前流行的一些开源的 API 网关。
	  
	  API 网关
	  
	  支持的认证和授权策略
	  
	  支持的服务发现组件
	  
	  支持的协议
	  
	  控制平面配置分发方法
	  
	  支持的插件扩展机制
	  
	  基础组织隶属
	  
	  Envoy Gateway
	  
	  OAuth2, JWT, mTLS, OIDC
	  
	  Kubernetes, EDS
	  
	  HTTP, HTTPS, gRPC
	  
	  xDS
	  
	  基于 Envoy Filter 的
	  
	  CNCF
	  
	  Kuma
	  
	  mTLS, JWT
	  
	  Kubernetes, Consul
	  
	  HTTP, HTTPS, gRPC, TCP
	  
	  REST, gRPC
	  
	  基于 Lua, Go 的
	  
	  CNCF ([View Highlight](https://read.readwise.io/read/01hxnx7jhgp8v9zm4esr6y6tq1))
	- Envoy Gateway 不仅优化了云原生时代的七层网关配置，而且为从边缘网关向服务网格过渡提供了一个平滑的道路。由于服务网格的推广面临一些挑战，如对应用的侵入性和运维团队推动问题，边缘网关则更易于被开发团队接受。Envoy Gateway 采用简化的 Kubernetes Gateway API，提高了流量管理和可观察性的能力。此外，Envoy Gateway 到 Istio 的过渡对于已熟悉 Envoy 功能的团队来说，将是一个自信的技术进步，同时还支持从标准的 Kubernetes Gateway API 到 Istio Ingress Gateway 的无缝切换，或者作为一个定制解决方案继续与 Istio 协作。这些特 ([View Highlight](https://read.readwise.io/read/01hxnxfvvfejtdm5ry8x3rxvvr))