title:: Containerd深度剖析-Nri篇 (highlights)
author:: [[知乎专栏]]
full-title:: "Containerd深度剖析-Nri篇"
category:: #articles
url:: https://zhuanlan.zhihu.com/p/595565298
![](https://picx.zhimg.com/v2-aaf11fd4f03b4cd023f7ad12225e80a7_720w.jpg?source=172ae18b)

- Highlights first synced by [[Readwise]] [[Jan 17th, 2024]]
	- kubelet ([View Highlight](https://read.readwise.io/read/01hmap0td5crtrb6p628cnycw8))
	- 在NRI中，下列容器元数据对插件是可用的。
	  
	    ID
	    pod ID
	    name
	    state
	    labels
	    annotations
	    command line arguments
	    environment variables
	    mounts
	    OCI hooks
	    linux
	      namespace IDs
	      devices
	      resources
	        memory
	          limit
	          reservation
	          swap limit
	          kernel limit
	          kernel TCP limit
	          swappiness
	          OOM disabled flag
	          hierarchical accounting flag
	          hugepage limits
	        CPU
	          shares
	          quota
	          period
	          realtime runtime
	          realtime period
	          cpuset CPUs
	          cpuset memory
	        Block I/O class
	        RDT class
	  
	  除了识别容器的数据外，这些信息还代表了容器的OCI规格中的相应数据。
	  
	  ***容器调整***
	  
	  在容器创建过程中，插件可以请求对以下容器参数进行更改。
	  
	    annotations
	    mounts
	    environment variables
	    OCI hooks
	    linux
	      devices
	      resources
	        memory
	          limit
	          reservation
	          swap limit
	          kernel limit
	          kernel TCP limit
	          swappiness
	          OOM disabled flag
	          hierarchical accounting flag
	          hugepage limits
	        CPU
	          shares
	          quota
	          period
	          realtime runtime
	          realtime period
	          cpuset CPUs
	          cpuset memory
	        Block I/O class
	        RDT class
	  
	  **容器更新**
	  
	  一旦一个容器被创建，插件可以请求对其进行更新。这些更新可以在响应另一个容器创建请求时请求，在响应任何容器更新请求时请求，在响应任何容器停止请求时请求，或者可以作为单独的非请求的容器更新请求的一部分请求。以下容器参数可以通过这种方式被更新。
	  
	    resources
	      memory
	        limit
	        reservation
	        swap limit
	        kernel limit
	        kernel TCP limit
	        swappiness
	        OOM disabled flag
	        hierarchical accounting flag
	        hugepage limits
	      CPU
	        shares
	        quota
	        period
	        realtime runtime
	        realtime period
	        cpuset CPUs
	        cpuset memory
	      Block I/O class
	      RDT class ([View Highlight](https://read.readwise.io/read/01hmamy98rq6hd8w8wxayaa4re))
	- 当前 kubelet 的实现是通过 cpuManager 的处理对象只能是 guaranteed 类的 pod， topologyManager 通过 cpuManager 提供的 hints 实现资源分配。
	  
	  kubelet 当前也不适合处理多种需求的扩展，因为在 kubelet 增加细粒度的资源分配会导致 kubelet 和 CRI 的界限越来越模糊。而上述 CRI 内的插件，则是在 CRI 容器生命周期期间调用，适合做 resoruce pinning 和节点的拓扑的感知。并且在 CRI 内部做插件定义和迭代，可以做到上层 kubernetes 以最小代价来适配变化。
	  
	  在容器生命周期中，CNI/NRI 插件能够注入到容器初始化进程的 Create 和 Start 之间：
	  
	  Create->NRI->Start
	  
	  以官方clearcfs示例：在启动容器前，依据 qos 类型调用 cgroup 命令，cpu.cfs_quota_us 为-1 表示不设上限。
	  
	  可以分析出 NRI 直接控制 cgroup，所以能有更底层的资源分配方式。不过越接近底层，处理逻辑的复杂度也越高 ([View Highlight](https://read.readwise.io/read/01hmamzqbwt22tb8ht16d6j5dj))