title:: 技术分享之 Ambient Mesh vs Sidecar vs Proxyless (highlights)
author:: [[Kubeservice博客]]
full-title:: "技术分享之 Ambient Mesh vs Sidecar vs Proxyless"
category:: #articles
url:: https://kubeservice.cn/2023/11/22/istio-ambient-mesh/
summary:: Comparison of Ambient Mesh, Sidecar, and Proxyless for implementing service mesh capabilities, with Ambient Mesh being a Sidecar-less mode providing shared proxy through ztunnel. Ambient Mesh offers non-intrusive application integration and uses components like ztunnel and Waypoint proxy to handle network functions. Sidecar and Proxyless modes have limitations in terms of control, complexity, and language dependencies compared to the evolving Ambient Mesh approach.
![](https://readwise-assets.s3.amazonaws.com/media/uploaded_book_covers/profile_1160846/home-bg-jeep.jpg)

- Highlights first synced by [[Readwise]] [[Apr 10th, 2024]]
	- Proxyless模式
	  
	  ![proxyless](https://kubeservice.cn/img/mesh/proxyless_hu608a648f4be03972e6cdbeb6b9a79a51_123752_filter_1820478240072544532.png)
	  
	  `Proxyless 模式`有以下限制：
	  
	  •   Proxyless 默认是将istiod的 xds 实现在Framework中，需要开发多种语言的sdk. 和开发框架 开发`语言绑定`
	  •   可移植性低: 无法通过切换sidecar的形式来非侵入式地升级基础设施. ([View Highlight](https://read.readwise.io/read/01hv1png1tczq2yhms1v0c22yw))
	- Ambient Mesh模式
	  
	  ![Ambient](https://kubeservice.cn/img/mesh/ambient_hu0d7d8e6d3d28b85d5cd2d2e9304f6057_79301_filter_14477136107647444095.png)
	  
	  •   `ambient 模式` :使得 Istio 越来越复杂，用户理解起来更加费力；控制平面为了支持多种数据平面部署模式，其实现将更加复杂。
	  •   安全问题: 统一的ztunnel，Node节点共用网络外围组件，扩大是故障影响半径；
	  •   升级 sidecar 也会带来很大的运营成本 ([View Highlight](https://read.readwise.io/read/01hv1pptgj553fvz11mn585p8x))