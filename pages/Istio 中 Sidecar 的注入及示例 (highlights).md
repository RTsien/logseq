title:: Istio 中 Sidecar 的注入及示例 (highlights)
author:: [[宋净超（Jimmy Song）]]
full-title:: "Istio 中 Sidecar 的注入及示例"
category:: #articles
url:: https://jimmysong.io/kubernetes-handbook/usecases/sidecar-spec-in-istio.html
![](https://readwise-assets.s3.amazonaws.com/static/images/article1.be68295a7e40.png)

- Highlights first synced by [[Readwise]] [[Dec 11th, 2023]]
	- 为了成为 服务网格中的一部分，Kubernetes 集群中的每个 Pod 都必须满足如下条件，这些规范不是由 Istio 自动注入的，而需要 生成 Kubernetes 应用部署的 YAML 文件时需要遵守的：
	  
	  1.  **Service 关联**：每个 pod 都必须只属于某**一个** [Kubernetes Service](https://kubernetes.io/docs/concepts/services-networking/service/) （当前不支持一个 pod 同时属于多个 service）。
	  2.  **命名的端口**：Service 的端口必须命名。端口的名字必须遵循如下格式 `<protocol>[-<suffix>]`，可以是 `http`、`http2`、 `grpc`、 `mongo`、 或者 `redis` 作为 `<protocol>` ，这样才能使用 Istio 的路由功能。例如 `name: http2-foo` 和 `name: http` 都是有效的端口名称，而 `name: http2foo` 不是。如果端口的名称是不可识别的前缀或者未命名，那么该端口上的流量就会作为普通的 TCP 流量来处理（除非使用 `Protocol: UDP` 明确声明使用 UDP 端口）。
	  3.  **带有 app label 的 Deployment**：我们建议 Kubernetes 的`Deploymenet` 资源的配置文件中为 Pod 明确指定 `app`label。每个 Deployment 的配置中都需要有个与其他 Deployment 不同的含有意义的 `app` label。`app` label 用于在分布式追踪中添加上下文信息。
	  4.  **Mesh 中的每个 pod 里都有一个 [Sidecar](https://jimmysong.io/kubernetes-handbook/GLOSSARY.html#sidecar)**：最后，网格中的每个 pod 都必须运行与 Istio 兼容的 [sidecar](https://jimmysong.io/kubernetes-handbook/GLOSSARY.html#sidecar)。以下部分介绍了将 [sidecar](https://jimmysong.io/kubernetes-handbook/GLOSSARY.html#sidecar) 注入到 pod 中的两种方法：使用`istioctl` 命令行工具手动注入，或者使用 Istio Initializer 自动注入。注意 [sidecar](https://jimmysong.io/kubernetes-handbook/GLOSSARY.html#sidecar) 不涉及到流量，因为它们与容器位于同一个 pod 中。 ([View Highlight](https://read.readwise.io/read/01hhcazq3evzqvbvpmwwq4stf9))