title:: 天穹云原生场景下的热迁移方案 (highlights)
author:: [[woa.com]]
full-title:: "天穹云原生场景下的热迁移方案"
category:: #articles
url:: https://km.woa.com/articles/show/618961?kmref=search&from_page=1&no=1
![](https://readwise-assets.s3.amazonaws.com/static/images/article3.5c705a01b476.png)

- Highlights first synced by [[Readwise]] [[Jul 7th, 2025]]
	- 天穹云原生通过和架平内核团队合作定制化内核、定制化容器运行时，基于单机 CRIU 方案，实现了进程级别的热迁移能力。在该基础上，将该方案与我们的云原生调度底座峰峦深度结合，实现了容器级别的热迁移。该方案主要有如下特点
	  
	  1.  与调度深度结合，用户无感知：用户可以主动设置业务的重调度方案，资源管理员也可以设置节点侧故障检测与驱逐方案。当容器被驱逐时，天穹云原生平台不是杀死容器并重新申请，而是直接将容器信息搬迁至正常节点。
	  
	  2.  保存Pod/容器内部全部信息：包含业务的全部状态信息和运行时信息，比如打开的文件句柄、socket等内部状态以及IP地址、容器文件系统等外部状态。
	  
	  3.  业务逻辑无需更改：业务侧无需修改业务代码，整个方案集成在平台层，最多需要引擎做少量资源申请的适配工作。
	  
	  4.  迁移后 ip 不变：在天穹跨集群 overlay 网络下，容器可以做到迁移前后容器 ip 不变。
	  
	  5.  不受物理集群限制：天穹业务运行在峰峦联邦集群架构下，支持跨物理集群统一调度，从而能够支持容器跨物理集群迁移。
	  
	  该方案目前在公司内多个场景落地，支持用户资源异常的情况下无感迁移，为用户在资源 sla 低场景下提升业务稳定性提供了强有力的支撑。下面我们将介绍几个热迁移的典型场景。 ([View Highlight](https://read.readwise.io/read/01jzj0qtvr0xwabmtmgfxj81ym))