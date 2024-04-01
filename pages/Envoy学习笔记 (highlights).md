title:: Envoy学习笔记 (highlights)
author:: [[woa.com]]
full-title:: "Envoy学习笔记"
category:: #articles
url:: https://iwiki.woa.com/p/1835112381
summary:: The text discusses the routing mechanism of stateful services using Envoy within Istio. It explains how Istio's control plane and data plane work together, with the control plane translating resources like virtual services and destination rules into Envoy configurations for routing traffic. It delves into Envoy concepts such as listeners, routes, and clusters, as well as the use of filters for traffic handling. The text also covers dynamic resources in Envoy, xDS protocol for configuration discovery, and various types of Discovery Services. Additionally, it details the configuration of Envoy within Istio, including dynamic resource fetching and listener setup. It further explores how EnvoyFilters are used to extend Envoy configurations for specific routing needs, highlighting the caution needed when using this feature. The text also introduces the use of Lua scripts within Envoy's HTTP filters for traffic manipulation and provides examples of Lua scripting for per-route configurations. Lastly, it discusses how E...
![](https://readwise-assets.s3.amazonaws.com/static/images/article2.74d541386bbf.png)

- Highlights first synced by [[Readwise]] [[Mar 26th, 2024]]
	- EnvoyFilter 是 istio 提供的一个 crd 资源， 它提供了一种机制来定制 Istio Pilot 生成的 Envoy 配置，当 virtual service 不再满足我们要求的时候。使用 EnvoyFilter 来修改某些字段的值，添加特定的过滤器，甚至添加全新的 listener、cluster 等。
	  
	  官网上说这个功能必须谨慎使用，因为不正确的配置可能破坏整个网格的稳定性。 ([View Highlight](https://read.readwise.io/read/01hswrtt389p76sct5ymv6hpnf))
	- 我们目前有状态服务的路由是采用的 Global Lua 级别的方式做了扩展，例如 zonesvr、gatesvr、guildsvr。我们去看我们集群里 Envoy 的 listener 运行时的 http_filters 配置，就会发现里面已经嵌入了几个 Lua 脚本：
	  
	  istioctl pc listener zonesvr-20220608083145z-0.cschang-dev -o yaml > envoy-listener.yaml
	  
	  ![](https://iwiki.woa.com/tencent/api/attachments/s3/url?attachmentid=11163745)
	  
	  Lua 脚本地址：[https://git.woa.com/red/routing/blob/master/internal/controllers/sscontroller/filter.lua](https://git.woa.com/red/routing/blob/master/internal/controllers/sscontroller/filter.lua)
	  
	  可见路由逻辑都写在 Lua 里了，例如最下面的 zonesvr 路由逻辑，我这里起了两个 zonesvr 实例，然后大区10001、10002、20001指向了第一个 pod 实例，20002指向了第二个 pod 实例；由于这些 Lua脚本所有请求都会执行，所以里面判断了目的地 host，命中 zonesvr-mesh 的话才会往下走；然后通过 header 中的 x-gid 取前5位获得大区号，得到要路由的 pod 地址；然后再将这个地址替换到 header 头里的 x-envoy-original-dst-host 字段。
	  
	  这里比较关键了，这个头信息有什么用？
	  
	  上面说过，如果 service 是 headless 类型的话，映射到 Envoy 的 cluster 配置里会是 ORIGINAL_DST 类型，也就是直接转发到原始 dns 解析的地址。Envoy 提供了一个配置 useHttpHeader ，默认为 false，如果为 true 的话，那么转发时就会用这个头信息里的 ip 进行转发，这个就是目的。 ([View Highlight](https://read.readwise.io/read/01hswrzxm9h3xvvyqzc6fa32z5))