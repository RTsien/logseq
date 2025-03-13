title:: 跨网络命名空间管理套接字：探索 Linux 套接字的使用 (highlights)
author:: [[Jimmy Song]]
full-title:: "跨网络命名空间管理套接字：探索 Linux 套接字的使用"
category:: #articles
url:: https://jimmysong.io/blog/cross-network-namespace-socket/
summary:: 最近我在研究 Istio Ambient 模式中的透明流量劫持，过程中涉及了跨网络命名空间管理套接字的功能。在 Istio 官方的博客 Maturing Istio Ambient: Compatibility Across Various Kubernetes Providers and CNIs 中也提到了这一点，让我对 Linux 套接字 API 的这个功能产生了浓厚的兴趣。所以，我决定写下这篇博客，分享如何在 Linux 系统中跨网络命名空间管理套接字的细节。
网络命名空间简介
网络命名空间是一种 Linux 内核特性，可以把系统的网络资源（例如 IP 地址、路由表等）分割成多个独立的实例。这样每个实例就可以为不同的进程提供独立的网络环境。比如，Docker 使用网络命名空间为每个容器提供独立的网络栈，让它们之间的网络资源互相隔离。
通过网络命名空间，不同的进程可以有独立的网络配置，比如不同的 IP 地址和路由设置。但即使网络命名空间实现了隔离，Linux 的套接字 API 仍然可以让进程跨网络命名空间操作套接字。
跨网络命名空间管理套接字
在一个命名空间中运行的进程可以创建一个套接字，并将它放到另一个网络命名空间中，这让我们可以实现非常灵活的网络通信。比如，可以在一个特定的网络命名空间中创建监听套接字，让其他命名空间中的进程共享这个套接字。这种功能在容器编排和微服务架构中非常有用。
下面是一个简单的例子，使用 nc 命令创建套接字，使其绑定到指定的网络设备或命名空间。


Mermaid Diagram

使用 Docker 演示这一功能
因为我使用的 macOS 不支持 Linux 的网络命名空间，但可以使用 Docker Desktop 模拟相应的环境。下面是在 macOS 上使用 Docker 来演示的方法：


Mermaid Diagram



安装 Docker Desktop

下载并安装 Docker Desktop，可以在 macOS 上运行 Linux 容器。
启动 Docker Desktop 后，我们可以在容器中模拟网络命名空间的相关操作。



配置虚拟网络

创建虚拟网络接口
docker network create --driver bridge my_bridge

为每个容器分配 IP 地址，以便它们可以相互通信。



创建网络命名空间（使用 Docker 容器模拟）


使用 Docker 创建两个容器，分别模拟两个网络命名空间：
docker run -d --name ns1 --network my_bridge --privileged alpine sleep infinity
docker run -d --name ns2 --network my_bridge --privileged alpine sleep infinity


创建容器时直接将它们连接到这个虚拟网络，以便它们可以相互通信。




跨命名空间创建套接字
使用 docker exec 命令进入容器并配置网络接口：


在 ns1 容器中运行一个监听套接字：
docker exec ns1 sh -c "nc -l -k -p 8080"


获取 ns1 容器的 IP 地址：
export NS1_IP=$(docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' ns1)


在 ns2 容器中，使用 nc 命令访问 ns1 容器中的套接字：
docker exec ns2 sh -c "echo 'Hello from ns2' | nc $NS1_IP 8080"


此时，我们可以在 ns1 容器的界面中看到来自 ns2 容器的 Hello from ns2 字符串。尽管 ns1 和 ns2 属于不同的容器，但通过正确的配置，ns2 仍然可以访问 ns1 中的套接字。




可以理解为通过在 ns2 监听 ns1 的套接字，类似于建立了一个“隧道”来实现通信。这种方式实际上是建立了一条直接的通信通道，使得两个容器之间能够进行数据交换。虽然它没有真正构建 VPN 那样复杂的隧道，但从逻辑上来说，ns2 和 ns1 之间可以通过这个套接字来传递数据，相当于建立了一个轻量级的点对点连接通道。
实际应用场景
这种“隧道”式的通信在许多实际场景中非常有用，以下是一些例子：

透明代理和负载均衡：通过套接字隧道可以将客户端请求转发到服务容器中，常用于透明代理或负载均衡。Istio、Envoy Proxy 和 HAProxy 等工具利用类似机制来管理服务间流量。
跨容器日志收集：通过套接字隧道，...
![](https://jimmysong.io/images/favicon.png)

- Highlights first synced by [[Readwise]] [[Nov 12th, 2024]]
	- 通过网络命名空间，不同的进程可以有独立的网络配置，比如不同的 IP 地址和路由设置。但即使网络命名空间实现了隔离，Linux 的套接字 API 仍然可以让进程跨网络命名空间操作套接字。
	  
	  在一个命名空间中运行的进程可以创建一个套接字，并将它放到另一个网络命名空间中，这让我们可以实现非常灵活的网络通信。比如，可以在一个特定的网络命名空间中创建监听套接字，让其他命名空间中的进程共享这个套接字。这种功能在容器编排和微服务架构中非常有用。 ([View Highlight](https://read.readwise.io/read/01jca6mjgxphaaykb0ks5ht9n5))