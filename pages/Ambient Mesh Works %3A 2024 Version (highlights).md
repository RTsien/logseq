title:: Ambient Mesh Works : 2024 Version (highlights)
author:: [[Yanick.xia]]
full-title:: "Ambient Mesh Works : 2024 Version"
category:: #articles
url:: https://blog.yanick.site/2024/04/08/networking/istio/ambient-mesh/ambient-mesh-2024/
summary:: The text discusses updates made in the 2024 version of Ambient Mesh Works, focusing on CNI & Ztunnel improvements like a new routing method and simplified traffic flow. It explains the logic behind CNI-CLI and CNI-Daemonset-POD components, detailing the process of redirecting traffic and handling Pod events. The text also covers the flow of traffic in different scenarios, emphasizing the role of Ztunnel in managing communication between Pods.
![](https://readwise-assets.s3.amazonaws.com/media/uploaded_book_covers/profile_1160846/image.8kzv8cluxv.webp)

- Highlights first synced by [[Readwise]] [[Apr 13th, 2024]]
	- CNI & Ztunnel
	  
	  ![image](https://cdn.jsdelivr.us/gh/yanickxia/picx-images-hosting@master/20240409/image.8hg9bp6iyx.webp)
	  
	  相较于之前的版本中，使用了标记路由的方式进行流量透传  
	  `Pod` -> `Host` -> `Ztunnel` 这样的路径，这样的路径对当前的其他的 `CNI` 比如 `flannel` 之类的冲击都比较的大，因此在当前的版本中，进行了较大的修改
	  
	  这里采用了一个的新的设计模式
	  
	  `Pod` -> `Ztunnel` 减少了到 `Host` 绕一圈的情况。
	  
	  这里实现了标准的 `CNI` 的接口，不熟悉的同学可以参考 [Kubernetes Network Plugins](https://blog.yanick.site/2022/03/01/kubernetes/cni/)
	  
	  所以这里需要实现 `2` 个东西
	  
	  •   `CNI-CLI`: 给 `kube` 来调用，`istio` 中的名称为 `istio-cni`
	  •   `CNI-Daemonset-POD`: 处理逻辑，`istio` 中的名称为 `install-cni`  
	    那我们就按顺序来看一下 ([View Highlight](https://read.readwise.io/read/01hv99b6t619fgfy1bcwp8b8xc))
	- 到这里，针对 POD 的逻辑我们已经做完了，下面就是 Ztunel 的逻辑了，我们简要的总结一下
	  
	  Created with Raphaël 2.2.0CNI工作机制KubeletKubeletCLICLIInstallInstallPodNSPodNS ([View Highlight](https://read.readwise.io/read/01hv99tna5g89pxm492aw7fr8f))
	- 实际上并不存在 `Ztunnel` -> `Ztunnel` 的直接路径，这个和之前的版本是不一样的，因此如果只从宿主机的角度看请求，是无法感知到这个请求的存在的，只有 `Ztunnel` -> `Pod` 的路径 ([View Highlight](https://read.readwise.io/read/01hv9asj1vadb0t7ehqdqs6672))