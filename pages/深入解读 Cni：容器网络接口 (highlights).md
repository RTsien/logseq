title:: 深入解读 Cni：容器网络接口 (highlights)
author:: [[Jimmy Song - 专注于探索后 Kubernetes 时代的云原生新范式 – 博客]]
full-title:: "深入解读 Cni：容器网络接口"
category:: #articles
url:: https://jimmysong.io/blog/cni-deep-dive/
summary:: 在容器化环境中，有效管理网络是至关重要的。容器网络接口（CNI）是一个标准，定义了容器应如何配置网络。本文将深入探讨 CNI 的基础知识，并带你了解 CNI 与 CRI 的关系。
什么是 CNI？
CNI（容器网络接口）规范为容器运行时和网络插件之间提供了一个通用的接口，旨在实现容器网络配置的标准化。
CNI 规范包含以下几个核心组成部分：

网络配置的格式：定义了管理员如何定义网络配置。
请求协议：描述了容器运行时如何向网络插件发出网络配置或清理请求。
插件执行过程：详细阐述了插件如何根据提供的配置执行网络设置或清理。
插件委派：允许插件将特定功能委托给其他插件执行。
结果返回：定义了插件执行完成后如何向运行时返回结果的数据格式。

CNI 规范通过定义这些核心组成部分，确保了不同的容器运行时和网络插件能够以一致的方式进行交互，实现网络配置的自动化和标准化。


  CNI 规范的一些要点



CNI 是一个插件化的容器化网络解决方案
CNI 插件为可执行文件
单个 CNI 插件的职责是单一的
CNI 插件是呈链式调用的
CNI 规范为一个容器定义一个 Linux 网络命名空间
CNI 的网络定义存储为 JSON 格式
网络定义通过 STDIN 输入流传输到插件，这意味着宿主机上不会存储网络配置文件，其他的配置参数通过环境变量传递给插件



CNI 插件根据操作类型，接收相应的网络配置参数，执行网络配置或清理任务，并返回执行结果。这一流程确保了容器网络的动态配置与容器生命周期的同步。
下图展示了 CNI 包含了众多的网络插件。
graph TB
    CR[Container Runtime] --> CNI["Container Network Interface (CNI)"]
    CNI --> LP[Loopback Plugin]
    CNI --> BP[Bridge Plugin]
    CNI --> PTP[PTP Plugin]
    CNI --> MACV[MACvlan Plugin]
    CNI --> IPV[IPvlan Plugin]
    CNI --> TPP[3rd-Party Plugin]

根据 CNI 规范
，一个 CNI 插件负责以某种方式配置容器的网络接口。插件可分为两大类：

“接口"插件，负责在容器内部创建网络接口并确保其具有连通性。
“...
![](https://jimmysong.io/blog/images/favicon-144.png)

- Highlights first synced by [[Readwise]] [[Apr 22nd, 2024]]
	- CNI（容器网络接口）规范为容器运行时和网络插件之间提供了一个通用的接口，旨在实现容器网络配置的标准化。
	  
	  CNI 规范包含以下几个核心组成部分：
	  
	  •   **网络配置的格式**：定义了管理员如何定义网络配置。
	  •   **请求协议**：描述了容器运行时如何向网络插件发出网络配置或清理请求。
	  •   **插件执行过程**：详细阐述了插件如何根据提供的配置执行网络设置或清理。
	  •   **插件委派**：允许插件将特定功能委托给其他插件执行。
	  •   **结果返回**：定义了插件执行完成后如何向运行时返回结果的数据格式。
	  
	  CNI 规范通过定义这些核心组成部分，确保了不同的容器运行时和网络插件能够以一致的方式进行交互，实现网络配置的自动化和标准化。 ([View Highlight](https://read.readwise.io/read/01hw21jd1tb67c699fwqgf3yb1))