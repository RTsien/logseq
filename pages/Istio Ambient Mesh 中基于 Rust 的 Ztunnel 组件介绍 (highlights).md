title:: Istio Ambient Mesh 中基于 Rust 的 Ztunnel 组件介绍 (highlights)
author:: [[宋净超]]
full-title:: "Istio Ambient Mesh 中基于 Rust 的 Ztunnel 组件介绍"
category:: #articles
url:: https://lib.jimmysong.io/blog/rust-based-ztunnel/
summary:: The text introduces Rust-Based Ztunnel for Istio Ambient Mesh, a lightweight proxy designed to enhance performance and simplify Istio's ambient mesh. Ztunnel focuses on functions like mTLS, authentication, L4 authorization, and telemetry for workloads without terminating HTTP traffic, offering a more efficient and secure way to connect and verify workloads in the Ambient Mesh environment. The Rust-based Ztunnel architecture, designed for low resource footprint and high performance, provides a specialized and efficient alternative to using Envoy, offering improved configurability and reduced overhead.
![](https://lib.jimmysong.io/media/sharing.png)

- Highlights first synced by [[Readwise]] [[Apr 10th, 2024]]
	- 为什么不重用 Envoy？[](https://lib.jimmysong.io/blog/rust-based-ztunnel#为什么不重用-envoy)
	  
	  当 Istio Ambient Mesh 于 2022 年 9 月 7 日发布时，ztunnel 是使用 Envoy 代理实现的。鉴于我们将 Envoy 用于 Istio 的其余部分——sidecar、网关和 Waypoint Proxy——我们开始使用 Envoy 实施 ztunnel 是很自然的选择。
	  
	  然而，我们发现虽然 Envoy 非常适合其他用例，但在 Envoy 中实现 ztunnel 具有挑战性，因为许多权衡、要求和用例与 sidecar 代理或入口网关有很大不同。此外，大多数使 Envoy 非常适合其他用例的东西，例如其丰富的 L7 功能集和可扩展性，都浪费在不需要这些功能的 ztunnel 中。 ([View Highlight](https://read.readwise.io/read/01hv1gz2mkgcrdw20eyqh4t785))
	- 专门构建的 ztunnel[](https://lib.jimmysong.io/blog/rust-based-ztunnel#专门构建的-ztunnel)
	  
	  在 Envoy 因我们的需求而失败后，我们开始考虑构建 ztunnel 的专用实现。我们的假设是，通过从一开始就考虑一个单一的重点用例进行设计，我们可以开发一个比将通用项目塑造成自定义用例更简单、性能更高的解决方案。使 ztunnel 简单化的明确决定是这一假设的关键；例如，类似的逻辑不适用于具有大量支持功能和集成的重写网关。
	  
	  这个专门建造的 [[ztunnel]] 涉及两个关键领域：
	  
	  •   ztunnel 与其 Istiod 之间的配置协议
	  •   ztunnel 的运行时实现 ([View Highlight](https://read.readwise.io/read/01hv1gznrrdvxxachhnhha6n0f))
	- 为了使 ztunnel 快速、安全和轻量级，[Rust](https://www.rust-lang.org/) 是一个显而易见的选择。然而，这不是我们第一次。鉴于 Istio 目前对 Go 的广泛使用，我们曾希望我们可以使基于 Go 的实现满足这些目标。在最初的原型中，我们构建了一些基于 Go 的实现和一个简单版本的基于 Rust 的实现。从我们的测试中，我们发现基于 Go 的版本不满足我们的性能和占用空间要求。虽然我们可能会进一步优化它，但我们认为基于 Rust 的代理从长远来看将为我们提供最佳实现。 ([View Highlight](https://read.readwise.io/read/01hv1h9mgn6h2fx816fs6mwsvz))