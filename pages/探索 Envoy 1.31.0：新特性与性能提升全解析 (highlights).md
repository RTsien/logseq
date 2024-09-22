title:: 探索 Envoy 1.31.0：新特性与性能提升全解析 (highlights)
author:: [[Jimmy Song - 专注于探索后 Kubernetes 时代的云原生新范式 – 博客]]
full-title:: "探索 Envoy 1.31.0：新特性与性能提升全解析"
category:: #articles
url:: https://jimmysong.io/blog/exploring-envoy-1-31-features-performance/
summary:: 今天 Envoy Proxy 1.31.0 发布，这是今年继 1.29、1.30 以来发布的第三个大版本。Envoy Proxy 1.31.0 的发布标志着此开源网络代理项目在性能优化和功能增强方面又迈出了重要一步。此版本包括了一系列引人注目的新特性、行为变化和新配置选项，下面我们将逐一解析这些更新，帮助你充分利用 Envoy 的最新能力。
新特性

HTTP/3 “Happy Eyeballs” 特性: HTTP/3的支持现在更为智能，新加入的“happy eyeballs”算法可以在多个 IP 地址族中更快找到最优连接路径，提升连接的可靠性和速度。
Proxy Protocol 类型元数据支持: 在代理协议监听器中，默认现在会填充类型化的元数据，为高级路由和策略实施提供更多的灵活性和精确控制。
Redis 命令支持: Envoy 现在支持所有 Bloom 1.0.0 的 Redis 命令，扩展了与 Redis 交互的能力，尤其适用于需要高级数据结构操作的场景。



    Happy Eyeballs
  

Happy Eyeballs（快乐的眼睛）是一种网络算法，主要用于当一个设备同时支持 IPv4 和 IPv6 时，快速决定应该使用哪种 IP 协议来建立连接。该算法通过几乎同时启动两个连接尝试——一个使用 IPv4，另一个使用 IPv6——并使用哪个首先成功建立的连接，从而减少了连接延迟。
在 HTTP/3 中应用 Happy Eyeballs 特性，尤其是 Envoy 1.31 版本中的实现，可以改进服务在支持多种网络协议的环境中的表现。例如，如果一个服务的 IPv4 连接速度比 IPv6 快，Envoy 会偏好 IPv4，反之亦然。这样做的好处是减少了尝试连接的总时间，提高了用户体验和服务效率。


行为变化

Thread Local 存储变更: SlotImpl 类的行为更新，其析构函数现可在任何线程上被调用，提高了线程局部存储的灵活性。
HTTP/2 和 QUIC 性能提升: 默认启用新的 HTTP/2 编解码器和对 HTTP/3 的优化，包括性能改进和新的连接尝试机制，显著提升了性能和稳定性。

弃用与移除
此版本中，多个旧有的配置和运行时标志被正式弃用和移除，以清理代码库并提升维护效率。这包括一些老旧的 TLS 和 HTTP 配置选项，用户应检查并更新他们的配置以免受到影响。
结论
Envoy...
![](https://jimmysong.io/blog/images/favicon-144.png)

- Highlights first synced by [[Readwise]] [[Sep 10th, 2024]]
	- Happy Eyeballs（快乐的眼睛）是一种网络算法，主要用于当一个设备同时支持 IPv4 和 IPv6 时，快速决定应该使用哪种 IP 协议来建立连接。该算法通过几乎同时启动两个连接尝试——一个使用 IPv4，另一个使用 IPv6——并使用哪个首先成功建立的连接，从而减少了连接延迟。 ([View Highlight](https://read.readwise.io/read/01j693snnfnxmc4e5vcxfaw24w))
		- 💡: 快乐眼球，哪个先成功用哪个