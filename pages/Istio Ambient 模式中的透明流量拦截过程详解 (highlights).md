title:: Istio Ambient 模式中的透明流量拦截过程详解 (highlights)
author:: [[JIMMY SONG]]
full-title:: "Istio Ambient 模式中的透明流量拦截过程详解"
category:: #articles
url:: https://jimmysong.io/blog/istio-ambient-traffic-interception/
summary:: 本文介绍了 Istio ambient 模式下的透明流量拦截过程，该模式允许服务网格在无需注入 sidecar 的情况下运行。通过 Istio CNI Node Agent 和 ztunnel 的协作，流量可以在不修改应用程序代码的情况下被拦截和重定向。该模式提升了兼容性、简化了运维，并通过 HBONE 协议实现了安全的加密传输。
![](https://jimmysong.io/images/favicon.png)

- Highlights first synced by [[Readwise]] [[Jan 31st, 2025]]
	- Istio CNI Node Agent[](https://jimmysong.io/blog/istio-ambient-traffic-interception/#istio-cni-node-agent)
	  
	  **Istio CNI Node Agent** 是 ambient 模式中的核心组件之一，负责在 Kubernetes 节点上检测加入 ambient 网格的 pod，并为这些 pod 配置流量重定向规则。需要注意的是，这里使用的是 Istio CNI Node Agent，而非传统的 Istio CNI 插件。Node Agent 是一个守护进程，与 ztunnel 协同工作，而不是直接参与网络插件的工作。 ([View Highlight](https://read.readwise.io/read/01jjttt806qkmzd68jtgv0zvc1))
	- ztunnel[](https://jimmysong.io/blog/istio-ambient-traffic-interception/#ztunnel)
	  
	  **ztunnel** 是 ambient 模式中的重要组件，以 DaemonSet 的形式运行在每个节点上，负责：
	  
	  •   接收并处理被重定向的流量；
	  •   实现 L4 层的策略，如 mTLS 加密和访问控制；
	  •   与控制平面通信以获取配置和证书。
	  
	  HBONE（基于 HTTP 的隧道协议）[](https://jimmysong.io/blog/istio-ambient-traffic-interception/#hbone基于-http-的隧道协议)
	  
	  **HBONE（HTTP-Based Overlay Network Encapsulation）** 是 Istio 引入的协议，用于在 ztunnel 和 waypoint proxy 之间传输任意 TCP 流量。HBONE 利用 HTTP/2 和 HTTP/3 的多路复用及加密特性，提高通信效率和安全性。 ([View Highlight](https://read.readwise.io/read/01jjttvz6wnb8zw7s5nczrgdbx))
	- 流量拦截过程详解[](https://jimmysong.io/blog/istio-ambient-traffic-interception/#流量拦截过程详解)
	  
	  在 ambient 模式下，应用程序 pod 无需修改代码，也不需要注入 sidecar。流量拦截和重定向的主要过程发生在 **pod 的网络命名空间** 内部，这种方式避免了与底层 CNI 的冲突。以下是其步骤概览：
	  
	  ![image](https://jimmysong.io/blog/istio-ambient-traffic-interception/7b94ccedcde4f27f06d158187d7904e2.svg)
	  
	  Istio ambient 模式的流量拦截过程
	  
	  流量拦截详细步骤[](https://jimmysong.io/blog/istio-ambient-traffic-interception/#流量拦截详细步骤)
	  
	  1.  **pod 启动与网络配置**：
	    •   Kubernetes 创建 pod 时，通过 Container Runtime Interface（CRI）调用底层 CNI 插件（如 Calico、Cilium）为 pod 配置网络。
	    •   此时，pod 的网络命名空间（netns）已经建立。
	  2.  **Istio CNI Node Agent 配置流量重定向**：
	    •   Istio CNI Node Agent 监测到新 pod 被标记为 ambient 模式（通过标签 `istio.io/dataplane-mode=ambient`）。
	    •   进入 pod 的网络命名空间，设置 iptables 规则以拦截流量。
	    •   将网络命名空间的文件描述符（FD）传递给 ztunnel。
	  3.  **ztunnel 在 pod 网络命名空间中启动监听套接字**：
	    •   ztunnel 接收到网络命名空间的 FD，在其中启动监听套接字以处理重定向的流量。
	  4.  **透明流量拦截与处理**：
	    •   应用程序发出的流量被 pod 内的 iptables 规则拦截，并透明地重定向到 ztunnel。
	    •   ztunnel 对流量执行策略检查、加密等处理后转发到目标服务。
	    •   返回的响应流量通过 ztunnel 解密并返回给应用程序。 ([View Highlight](https://read.readwise.io/read/01jjtv6c34fwfxqvcdrws24ams))