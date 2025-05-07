title:: Kubernetes 集群伸缩组件 - Karpenter (highlights)
author:: [[ryan4yin]]
full-title:: "Kubernetes 集群伸缩组件 - Karpenter"
category:: #articles
url:: https://thiscute.world/posts/kubernetes-cluster-autoscaling-1-karpenter/
summary:: 前言Kubernetes 具有非常丰富的动态伸缩能力，这体现在多个层面： Workloads 的伸缩：通过 Horizontal Pod Autoscaler（HPA）和 Vertical Pod Autoscal
![](https://thiscute.world/apple-touch-icon.png)

- Highlights first synced by [[Readwise]] [[Apr 30th, 2025]]
	- Karpenter 则完全从零开始实现了一套节点管理系统，它直接管理所有节点（云服务器，如 AWS EC2），负责节点的创建、删除、修改等操作。
	  
	  相较 Cluster Autoscaler, Karpenter 的优势主要体现在以下几个方面：
	  
	  1.  **声明式地定义节点池**: Karpenter 提供了一套 CRD 来定义节点池，用户只需要编写好 Yaml 配置部署到集群中，Karpenter 就会根据配置自动申请与管理节点。这比 Cluster Autoscaler 的配置要方便得多。
	    •   以 AWS 为例，你简单地改几行 Yaml 配置，就可以修改掉节点池的实例类型、AMI 镜像、数量上下限、磁盘大小、节点 Labels 跟 Taints、EC2 Tags 等信息。
	    •   借助 Flux 或 ArgoCD 等 GitOps 工具，你还可以实现自动化的节点池管理以及配置的版本控制。
	  2.  **成本感知的节点管理**：Karpenter 不仅负责节点数量的伸缩，它还能根据节点的规格、负载情况、成本等因素来选择最优的节点类型，以达成成本、性能、稳定性之间的平衡。 - 具体而言，Karpenter 在成本优化方面具有这些 Cluster Autoscaler 不具备的功能：
	    •   **Spot/On-Demand 实例调整**: 在 AWS 上，Karpenter 可以设置为优先使用 Spot 实例，并在申请不到 Spot 实例时自动切换到 On-Demand 实例，从而大大降低成本。
	    •   **多节点类型支持**: Karpenter 支持在同一个集群中使用多种不同规格的节点，并且支持控制不同实例类型的优先级、数量或占比，以满足不同的业务需求。
	    •   **节点替换策略**：Karpenter 支持灵活的节点替换策略，可以通过 Yaml 控制每个节点池的节点替换条件、频率、比例等参数，以避免因节点替换导致的服务不可用。
	    •   **节点的生命周期管理**：Karpenter 支持定义节点的生命周期策略，可以根据节点的年龄、负载、成本等因素来决定节点的续租、下线、销毁等操作。而 Cluster Autoscaler 只能控制节点的数量，它不直接管理节点，也就做不到此类节点的精细管理。
	    •   **主动优化**：Karpenter 支持主动根据负载情况使用不同实例类型的节点替换高风险节点，或合并低负载节点，以节省成本。
	    •   **Pod 精细化调度**：Karpeneter 本身也是一个调度器，它能根据 Pod 的资源需求、优先级、Node Affinity、Topology Spread Constraints 等因素来申请节点并主动将 Pod 调度到该节点上。而 Cluster Autoscaler 只能控制节点的数量，并无调度能力。
	  3.  **快速、高效**：因为 Karpenter 直接创建、删除节点，并且主动调度 Pod，所以它的伸缩速度与效率要比 Cluster Autoscaler 高很多。这是因为 Karpneter 能快速获知节点创建、删除、加入集群是否成功，而 Cluster Autoscaler 只能被动地等待云厂商的伸缩组或节点池服务完成这些操作，它无法主动感知节点的状态。 ([View Highlight](https://read.readwise.io/read/01jt1gj020r5hbvspx9xnfbt8e))