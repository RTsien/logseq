title:: 使用 Istio 前需要考虑的问题 (highlights)
author:: [[宋净超（Jimmy Song）]]
full-title:: "使用 Istio 前需要考虑的问题"
category:: #articles
url:: https://jimmysong.io/kubernetes-handbook/usecases/before-using-istio.html
![](https://readwise-assets.s3.amazonaws.com/static/images/article1.be68295a7e40.png)

- Highlights first synced by [[Readwise]] [[Dec 4th, 2023]]
	- 使用 Istio 无法做到完全对应用透明
	  
	  服务通信和治理相关的功能迁移到 [sidecar](https://jimmysong.io/kubernetes-handbook/GLOSSARY.html#sidecar) 进程中后， 应用中的 SDK 通常需要作出一些对应的改变。
	  
	  比如 SDK 需要关闭一些功能，例如重试。一个典型的场景是，SDK 重试 m 次，[sidecar](https://jimmysong.io/kubernetes-handbook/GLOSSARY.html#sidecar) 重试 n 次，这会导致 m * n 的重试风暴，从而引发风险。
	  
	  此外，诸如 trace header 的透传，也需要 SDK 进行升级改造。如果你的 SDK 中还有其它特殊逻辑和功能，这些可能都需要小心处理才能和 Isito [sidecar](https://jimmysong.io/kubernetes-handbook/GLOSSARY.html#sidecar) 完美配合。 ([View Highlight](https://read.readwise.io/read/01hgsxvyy2p1rerq2nxxksvdm4))
- New highlights added [[Dec 4th, 2023]] at 3:35 AM
	- Istio 对非 Kubernetes 环境的支持有限
	  
	  在业务迁移至 Istio 的同时，可能并没有同步迁移至 Kubernetes，而还运行在原有 PAAS 系统之上。 这会带来一系列挑战：
	  
	  •   原有 PaaS 可能没有容器网络，Istio 的服务发现和流量劫持都可能要根据旧有基础设施进行适配才能正常工作；
	  •   如果旧有的 PAAS 单个实例不能很好的管理多个容器（类比 Kubernetes 的 Pod 和 Container 概念），大量 Istio [sidecar](https://jimmysong.io/kubernetes-handbook/GLOSSARY.html#sidecar) 的部署和运维将是一个很大的挑战；
	  •   缺少 Kubernetes webhook 机制，[sidecar](https://jimmysong.io/kubernetes-handbook/GLOSSARY.html#sidecar) 的注入也可能变得不那么透明，而需要耦合在业务的部署逻辑中； ([View Highlight](https://read.readwise.io/read/01hgszs2bc11qs3kqcvn3qcb7j))
	- 只有 HTTP 协议是一等公民
	  
	  Istio 原生对 HTTP 协议提供了完善的全功能支持，但在真实的业务场景中，私有化协议却非常普遍，而 Istio 却并未提供原生支持。
	  
	  这导致使用私有协议的一些服务可能只能被迫使用 TCP 协议来进行基本的请求路由，这会导致很多功能的缺失，这其中包括 Istio 非常强大的基于内容的消息路由，如基于 header、 path 等进行权重路由。 ([View Highlight](https://read.readwise.io/read/01hgszs8784ce20xj89x131b0p))
	- 扩展 Istio 的成本并不低
	  
	  虽然 Istio 的总体架构是基于高度可扩展而设计，但由于整个 Istio 系统较为复杂，如果你对 Istio 进行过真实的扩展，就会发现成本不低。
	  
	  以扩展 Istio 支持某一种私有协议为例，首先你需要在 Istio 的 API 代码库中进行协议扩展，其次你需要修改 Istio 代码库来实现新的协议处理和下发，然后你还需要修改 xds 代码库的协议，最后你还要在 Envoy 中实现相应的 Filter 来完成协议的解析和路由等功能。
	  
	  在这个过程中，你还可能面临上述数个复杂代码库的编译等工程挑战（如果你的研发环境不能很好的使用 Docker 或者无法访问部分国外网络的情况下）。
	  
	  即使做完了所有的这些工作，你也可能面临这些工作无法合并回社区的情况，社区对私有协议的扩展支持度不高，这会导致你的代码和社区割裂，为后续的升级更新带来隐患。 ([View Highlight](https://read.readwise.io/read/01hgt1280fkm78g65kgryx45re))
	- Istio 在集群规模较大时的性能问题
	  
	  Istio 默认的工作模式下，每个 [sidecar](https://jimmysong.io/kubernetes-handbook/GLOSSARY.html#sidecar) 都会收到全集群所有服务的信息。如果你部署过 Istio 官方的 Bookinfo 示例应用，并使用 Envoy 的 config dump 接口进行观察，你会发现，仅仅几个服务，Envoy 所收到的配置信息就有将近 20w 行。
	  
	  可以想象，在稍大一些的集群规模，Envoy 的内存开销、Istio 的 CPU 开销、XDS 的下发时效性等问题，一定会变得尤为突出。
	  
	  Istio 这么做一是考虑这样可以开箱即用，用户不用进行过多的配置，另外在一些场景，可能也无法梳理出准确的服务之间的调用关系，因此直接给每个 [sidecar](https://jimmysong.io/kubernetes-handbook/GLOSSARY.html#sidecar) 下发了全量的服务配置，即使这个 [sidecar](https://jimmysong.io/kubernetes-handbook/GLOSSARY.html#sidecar) 只会访问其中很小一部分服务。
	  
	  当然这个问题也有解法，你可以通过 [sidecar](https://jimmysong.io/kubernetes-handbook/GLOSSARY.html#sidecar) [CRD](https://jimmysong.io/kubernetes-handbook/GLOSSARY.html#crd) 来显示定义服务调用关系，使 Envoy 只得到他需要的服务信息，从而大幅降低 Envoy 的资源开销，但前提是在你的业务线中能梳理出这些调用关系。 ([View Highlight](https://read.readwise.io/read/01hgt139g2tfqc7g51z9c2k3t8))
	- XDS 分发没有分级发布机制
	  
	  当你对一个服务的策略配置进行变更的时候，XDS 不具备分级发布的能力，所有访问这个服务的 Envoy 都会立即收到变更后的最新配置。这在一些对变更敏感的严苛生产环境，可能是有很高风险甚至不被允许的。
	  
	  如果你的生产环境严格要求任何变更都必须有分级发布流程，那你可能需要考虑自己实现一套这样的机制。 ([View Highlight](https://read.readwise.io/read/01hgt13xk5qehvxa5869qwhnep))
	- Istio 组件故障时是否有退路？
	  
	  以 Istio 为代表的 [sidecar](https://jimmysong.io/kubernetes-handbook/GLOSSARY.html#sidecar) 架构的特殊性在于，[sidecar](https://jimmysong.io/kubernetes-handbook/GLOSSARY.html#sidecar) 直接承接了业务流量，而不像一些其他的基础设施那样，只是整个系统的旁路组件（比如 Kubernetes）。
	  
	  因此在 Isito 落地初期，你必须考虑，如果 [sidecar](https://jimmysong.io/kubernetes-handbook/GLOSSARY.html#sidecar) 进程挂掉，服务怎么办？是否有退路？是否能 fallback 到直连模式？
	  
	  在 Istio 落地过程中，是否能无损 fallback，通常决定了核心业务能否接入服务网格。 ([View Highlight](https://read.readwise.io/read/01hgt14rf02wkpe6gbj5g9q99f))
	- Istio 缺乏成熟的产品生态
	  
	  Istio 作为一套技术方案，却并不是一套产品方案。如果你在生产环境中使用，你可能还需要解决可视化界面、权限和账号系统对接、结合公司已有技术组件和产品生态等问题，仅仅通过命令行来使用，可能并不能满足你的组织对权限、审计、易用性的要求。
	  
	  而 Isito 自带的 Kiali 功能还十分简陋，远远没有达到能在生产环境使用的程度，因此你可能需要研发基于 Isito 的上层产品。目前有一些服务网格的商业化公司致力于解决 Istio 的产品生态问题，如 [Tetrate](https://tetrate.io) 就是在基于 Istio、Envoy 和 Apache SkyWalking 构建企业级服务网格。 ([View Highlight](https://read.readwise.io/read/01hgt15a7jk90mgbaz1s8ww2wm))
	- Istio 目前解决的问题域还很有限
	  
	  Istio 目前主要解决的是分布式系统之间服务调用的问题，但还有一些分布式系统的复杂语义和功能并未纳入到 Istio 的 [sidecar](https://jimmysong.io/kubernetes-handbook/GLOSSARY.html#sidecar) 运行时之中，比如消息发布和订阅、状态管理、资源绑定等等。
	  
	  云原生应用将会朝着多 [sidecar](https://jimmysong.io/kubernetes-handbook/GLOSSARY.html#sidecar) 运行时或将更多分布式能力纳入单 [sidecar](https://jimmysong.io/kubernetes-handbook/GLOSSARY.html#sidecar) 运行时的方向继续发展，以使服务本身变得更为轻量，让应用和基础架构彻底解耦。
	  
	  如果你的生产环境中，业务系统对接了非常多和复杂的分布式系系统中间件，Istio 目前可能并不能完全解决你的应用的云原生化诉求。 ([View Highlight](https://read.readwise.io/read/01hgt16jrreb1x5yvyfpv5wczc))