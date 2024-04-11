title:: 有状态服务组件架构 (highlights)
author:: [[woa.com]]
full-title:: "有状态服务组件架构"
category:: #articles
url:: https://iwiki.woa.com/p/4006861811
![](https://readwise-assets.s3.amazonaws.com/static/images/article0.00998d930354.png)

- Highlights first synced by [[Readwise]] [[Apr 11th, 2024]]
	- **1、coordinator:**
	  
	  是 zone 类型服务的控制器，用于创建 zone 类型服务以及处理大区区号 ShardID 和 zone 实例 Pod 的对应关系。
	  
	  ShardedSet 用于描述 zone 类型服务的 Workload，coordinator 接收到 ShardedSet 资源创建请求后会创建出对应的 zone 型服务实例 pod，并进行管理；
	  
	  ShardMapping 用于描述大区区号和 zone 型服务的对应关系，目前用于手动配置哪个区运行在哪个 zonesvr Pod 上。
	  
	  **2、sspilot:**
	  
	  用于处理流量路由，主要功能是接收 Route 数据源然后通过定义的路由规则生成 EnvoyFilter，最终作用在每个服务的 istio-proxy 中。主要关心 ShardedService 这个 CRD，以及 Service、Endpoints 资源。
	  
	  ShardedService 每个有状态服务的路由都需要有，用于定义转发规则和路由条目数据的来源。
	  
	  sspilot 有三个 controller 组成：
	  
	  ssController：用于监听 ShardedService 资源，并将里面的路由规则描述转化为 wasm 插件配置，最终生成 EnvoyFilter。
	  
	  podsController：用于监听 pod 资源，如果发现其是属于我们关心的（ShardedService 中定义的，例如 roomsvr），则会通知给 ssController 中的 podnameStrategy，用于更新路由条目数据。
	  
	  hsController：监听 Service 资源，如果是 Headless 类型的 Service，则会生成一个EnvoyFilter，用于将 Envoy 侧的一个配置 use_http_header 置为 true。 ([View Highlight](https://read.readwise.io/read/01hv5tm77zdyfbx91p4p49821h))
	- **关键组件：**
	  
	  **1、coordinator:**
	  
	  是 zone 类型服务的控制器，用于创建 zone 类型服务以及处理大区区号 ShardID 和 zone 实例 Pod 的对应关系。
	  
	  ShardedSet 用于描述 zone 类型服务的 Workload，coordinator 接收到 ShardedSet 资源创建请求后会创建出对应的 zone 型服务实例 pod，并进行管理；
	  
	  ShardMapping 用于描述大区区号和 zone 型服务的对应关系，目前用于手动配置哪个区运行在哪个 zonesvr Pod 上。
	  
	  **2、sspilot:**
	  
	  用于处理流量路由，主要功能是接收 Route 数据源然后通过定义的路由规则生成 EnvoyFilter，最终作用在每个服务的 istio-proxy 中。主要关心 ShardedService 这个 CRD，以及 Service、Endpoints 资源。
	  
	  ShardedService 每个有状态服务的路由都需要有，用于定义转发规则和路由条目数据的来源。
	  
	  sspilot 有三个 controller 组成：
	  
	  ssController：用于监听 ShardedService 资源，并将里面的路由规则描述转化为 wasm 插件配置，最终生成 EnvoyFilter。
	  
	  podsController：用于监听 pod 资源，如果发现其是属于我们关心的（ShardedService 中定义的，例如 roomsvr），则会通知给 ssController 中的 podnameStrategy，用于更新路由条目数据。
	  
	  hsController：监听 Service 资源，如果是 Headless 类型的 Service，则会生成一个EnvoyFilter，用于将 Envoy 侧的一个配置 use_http_header 置为 true。 ([View Highlight](https://read.readwise.io/read/01hv3t5bts1y3033m1ggm8faqm))