title:: Kubernetes V1.33：镜像拉取策略终于按你的预期工作了！ (highlights)
author:: [[Kubernetes]]
full-title:: "Kubernetes V1.33：镜像拉取策略终于按你的预期工作了！"
category:: #articles
url:: https://mp.weixin.qq.com/s/0HsIq1lWxjK71D38wbEF_Q
summary:: Kubernetes v1.33 修复了镜像拉取策略中存在十多年的安全问题，确保只有有权限的 Pod 能使用私有镜像。新版本中，kubelet 会验证每个 Pod 的凭据，防止未授权的 Pod 使用已存在的镜像。该特性目前为 Alpha 版本，未来将继续优化凭据管理和性能。
![](https://mmbiz.qpic.cn/mmbiz_jpg/GpkQxibjhkJyoARhxuBV99RFplS404t3LF193HyE5ltVnEEBwtoR1sViaiaWV434Paq0YeNdrricv5ia0c7kwpvTRlw/0?wx_fmt=jpeg)

- Highlights first synced by [[Readwise]] [[Jul 16th, 2025]]
	- 如果 **Namespace Y** 中的 **Pod B** 也被调度到 **Node 1**，就会出现意外（甚至是不安全）的情况。**Pod B** 可以引用同一个私有镜像，指定 `IfNotPresent` 镜像拉取策略。**Pod B** 未在其 `imagePullSecrets` 中引用 **Secret 1**（甚至未引用任何 Secret）。 当 kubelet 尝试运行此 Pod 时，它会采用 `IfNotPresent` 策略。 kubelet 发现本地已存在**镜像 Foo**，会将**镜像 Foo** 提供给 **Pod B**。 即便 **Pod B** 一开始并未提供授权拉取镜像的凭据，却依然能够运行此镜像。
	  
	  ![](https://mmbiz.qpic.cn/mmbiz_png/GpkQxibjhkJyoARhxuBV99RFplS404t3LKqqVS7xyFOICpwibBPYoFfMwGC6xPVTt6JQSbuANxrbZ9ansjfjAG2w/640?wx_fmt=png&from=appmsg) ([View Highlight](https://read.readwise.io/read/01k08n55xawq38mmf6c9q8mjpc))
		- 💡: 1.33解决了这个问题
	- 工作原理
	  
	  此特性基于每个节点上存在的持久化文件缓存。以下简要说明了此特性的工作原理。 完整细节请参见 KEP-2535[5]。
	  
	  首次请求某镜像的流程如下：
	  
	  1.  请求私有仓库中某镜像的 Pod 被调度到某节点。
	  2.  此镜像在节点上不存在。
	  3.  kubelet 记录一次拉取镜像的意图。
	  4.  kubelet 从 Pod 引用的 Kubernetes Secret 中提取凭据作为镜像拉取 Secret，并使用这些凭据从私有仓库拉取镜像。
	  5.  镜像已成功拉取后，kubelet 会记录这次成功的拉取。记录包括所使用的凭据细节（哈希格式）以及构成这些凭据的原始 Secret。
	  6.  kubelet 移除原始意图记录。
	  7.  kubelet 保留成功拉取的记录供后续使用。
	  
	  当以后调度到同一节点的 Pod 请求之前拉取过的私有镜像：
	  
	  1.  kubelet 检查新 Pod 为拉取镜像所提供的凭据。
	  2.  如果这些凭据的哈希或其源 Secret 与之前成功拉取记录的哈希或源 Secret 相匹配，则允许此 Pod 使用之前拉取的镜像。
	  3.  如果在该镜像的成功拉取记录中找不到这些凭据或其源 Secret，则 kubelet 将尝试使用这些新的凭据从远程仓库进行拉取，同时触发认证流程。 ([View Highlight](https://read.readwise.io/read/01k08n9nst4dtndzj182145ve3))