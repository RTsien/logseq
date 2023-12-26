title:: 基于 Rust-VMM 实现 K8s Runtime (highlights)
author:: [[tencent.com]]
full-title:: "基于 Rust-VMM 实现 K8s Runtime"
category:: #articles
url:: https://cloud.tencent.com/developer/article/2134593
![](https://cloudcache.tencentcs.com/open_proj/proj_qcloud_v2/gateway/shareicons/cloud.png)

- Highlights first synced by [[Readwise]] [[Dec 21st, 2023]]
	- 腾讯云在安全容器上融合和上述方案的优点，结合腾讯云在虚拟化，存储和网络方面的优势， 选择使用mVMd + QEMU + EKLET的方案，实现一个弹性的Kubernetes的服务，即EKS，大家可以访问一下腾讯云的官网，看一下EKS的介绍。 ([View Highlight](https://read.readwise.io/read/01hj5mq6hfdcaax9mgav4cq858))
	- EKS是基于Hypervisor的虚拟化的解决方案，不同于Kata，EKS使用的containerd + mVMd组件更加轻量，调用路径更短。通过containrtd + mVMd来实现对于上层K8s调用的CRI的解析，并且把它转化成真正对于底层一个一个Guest VM或者QEMU的控制指令，在guest VM里会启动相应的containers容器。
	  
	  在整个的架构中，在Runtime方面最大的瓶颈是QEMU，因为QEMU有几十年的历史了，所以存在着它比较臃肿，反应慢，占用的资源多等等问题。所以让QEMU作为底层Runtime还不够快，不够安全。为了增强QEMU的性能和安全特性，我们很自然把眼光投向了Rust-Vmm。 ([View Highlight](https://read.readwise.io/read/01hj5mrzq17018kcx4ekexga2t))