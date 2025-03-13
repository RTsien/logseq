title:: 使用 Envoy 实现 HTTP/2 CONNECT 隧道：原理与实践 (highlights)
author:: [[Jimmy Song]]
full-title:: "使用 Envoy 实现 HTTP/2 CONNECT 隧道：原理与实践"
category:: #articles
url:: https://jimmysong.io/blog/http2-envoy-tunnel-demo/
summary:: 在最近对 Istio Ambient 模式的研究中，我发现 HTTP2 Connect 方法被用作创建隧道的核心技术，以实现透明流量的拦截和转发。HTTP/2 CONNECT 隧道是一种强大的工具，可以在已有的 HTTP/2 连接中创建高效的隧道，用于传输原始的 TCP 数据。这篇文章通过一个简单的 Demo，展示了如何使用 Envoy 来实现 HTTP/2 CONNECT 隧道的基本功能。
什么是 HTTP2 Connect 方法以及 HBONE 隧道？
HTTP2 Connect 方法是一种标准化的方式来创建隧道，用于透明地传输数据。特别是在 Istio 的 Ambient 模式中，它为代理数据平面之间的通信提供了一种高效的手段。HBONE（HTTP-Based Overlay Network Environment）隧道则是基于这种 HTTP2 Connect 技术的实现，用于 Istio 中的透明流量拦截和转发。通过使用 HBONE，数据可以有效地通过 HTTP2 隧道安全传输，替代了传统的 Sidecar 模式。这一创新设计极大地简化了服务网格的管理和部署。
HBONE 是 Istio 特有的术语，它是一种安全隧道协议，用于在 Istio 组件之间进行通信。在当前的 Istio 实现中，HBONE 协议包含了三个开放标准：

HTTP/2
HTTP CONNECT
Mutual TLS (mTLS)

HTTP CONNECT 用于建立隧道连接，mTLS 用于安全地加密连接，而 HTTP/2 用于在单一安全隧道中多路复用应用连接流并传输附加的流级元数据。更多关于 HBONE 隧道的细节可以参考官方文档：HBONE 详细介绍。
使用 HTTP2 Connect 建立隧道的基本原理
HTTP2 Connect 方法允许我们创建一个类似于 VPN 的隧道，通过这个隧道可以安全地传递数据。建立隧道的基本步骤如下：

首先，客户端向代理发送一个普通的 TCP 或 HTTP 链接请求。
代理接收到请求后，代表客户端向目标服务器发送一个带有 CONNECT 方法的 HTTP2 请求。
如果服务器允许建立隧道，那么它会返回一个 HTTP2 200 OK 的响应给代理。
随后，客户端、代理和服务器之间的双向流数据就可以通过这个隧道进行传输。

这种方法能够使得数据的传输过程更加透明且安全，特别适用于需要高效通信和端到端加密的场景。
下图展示了 HTTP2 Connect 方法建立隧道的基本过程。


HTTP2 Connect 方法建立隧道的基本过程

Demo：使用 Envoy 与上游 Server 建立 HTTP/2 Connect 隧道
本示例展示了一个基础场景：

客户端：向 Envoy 代理发送文本消息。
Envoy：接收客户端的 TCP 数据，将其封装为 HTTP/2 CONNECT 请求，并与上游服务器建立加密隧道。
服务器：接收来自 Envoy 的 HTTP/2 CONNECT 流量，解封装并返回响应给客户端。

架构图如下：


架构图

我们将使用 Node.js 来编写客户端和服务端，并将服务端和 Envoy 代理运行在容器中，在本地通过客户端访问 Envoy 代理从而达到访问客户端的目的。
完整的目录结构如下：
envoy-http2-tunnel/
├── certs/
│   ├── openssl.cnf
│   ├── server.crt
│   ├── server.key
├── client/
│   └── client.js
├── docker-compose.yml
├── envoy.yaml
└── server/
    ├── Dockerfile
    └── server.js
核心功能展示
1. HTTP/2 CONNECT 隧道的基本实现

客户端通过普通的 TCP 连接与 Envoy 通信。
Envoy 将 TCP 数据封装为 HTTP/2 CONNECT 请求，发送到上游服务器。
服务器接收并解封装隧道中的数据，进行处理后返回响应。
隧道通信对客户端完全透明。

2. Envoy 的透明代理能力

Envoy 作为中间代理，将客户端与服务器之间的通信逻辑完全封装。
客户端无需支持复杂的协议（如 HTTP/2 或 TLS），Envoy 代理完成所有协议转换。

3. 加密通信的实现

Envoy 与服务器之间的通信通过 TLS 加密，确保隧道内的数据安全。
服务器终止 TLS，处理解密后的数据。

4. 隧道的简化使用场景

通过该 Demo，可以快速理解 HTTP/2 CONNECT 隧道的建立和基本数据传输流程。

环境准备
1. 安装 Node.js
确保...
![](https://jimmysong.io/images/favicon.png)

- Highlights first synced by [[Readwise]] [[Mar 3rd, 2025]]
	- HTTP2 Connect 方法是一种标准化的方式来创建隧道，用于透明地传输数据。特别是在 Istio 的 Ambient 模式中，它为代理数据平面之间的通信提供了一种高效的手段。HBONE（HTTP-Based Overlay Network Environment）隧道则是基于这种 HTTP2 Connect 技术的实现，用于 Istio 中的透明流量拦截和转发。通过使用 HBONE，数据可以有效地通过 HTTP2 隧道安全传输，替代了传统的 Sidecar 模式。这一创新设计极大地简化了服务网格的管理和部署。
	  
	  HBONE 是 Istio 特有的术语，它是一种安全隧道协议，用于在 Istio 组件之间进行通信。在当前的 Istio 实现中，HBONE 协议包含了三个开放标准：
	  
	  •   **HTTP/2**
	  •   **HTTP CONNECT**
	  •   **Mutual TLS (mTLS)** ([View Highlight](https://read.readwise.io/read/01jnd8dz50gskep4wfp0x834pe))