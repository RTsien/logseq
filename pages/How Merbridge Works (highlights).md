title:: How Merbridge Works (highlights)
author:: [[Yanick.xia]]
full-title:: "How Merbridge Works"
category:: #articles
url:: https://blog.yanick.site/2023/05/17/networking/istio/how-merbridge-works/
summary:: 在 服务网格加速器 Merbridge 正式进入 CNCF 沙箱 中提及使用 Cillum 来提速 Istio，我们来瞧瞧这个原理是如何的。      注意     本文基于 0.8.1 版本
![](https://jsd.cdn.zzko.cn/gh/yanickxia/picture-bed@master/2024/202403132227792.png)

- Highlights first synced by [[Readwise]] [[Jun 20th, 2025]]
	- 其中，之所以在 connect 时，修改目的地址为 127.x.y.z 而不是 127.0.0.1，是因为在不同的 Pod 中，可能产生冲突的四元组，使用此方式即可巧妙地避开冲突 (每个 Pod 间的目的 IP 不同，不会出现冲突的情况)。 ([View Highlight](https://read.readwise.io/read/01jy4fxk9r66tpb4w2sghnxanm))