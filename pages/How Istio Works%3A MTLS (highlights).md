title:: How Istio Works: MTLS (highlights)
author:: [[Yanick.xia]]
full-title:: "How Istio Works: MTLS"
category:: #articles
url:: https://blog.yanick.site/2024/04/01/networking/istio/how-it-works/mtls/istio-mtls/#more
summary:: 今天我们来瞧瞧 Istio 是如何实现 mTLS 的。
![](https://jsd.cdn.zzko.cn/gh/yanickxia/picx-images-hosting@master/20240319/image.73tpdfuh3i.webp)

- Highlights first synced by [[Readwise]] [[Apr 10th, 2024]]
	- 对于服务端来说，处理起来是比较简单的，我们只需要关注 `CreateCertificate` 函数的逻辑即可。这里其实只有2个步骤
	  
	  1.  通过 `Authenticators` 来判断来源是否合法，以及其 `Identities` 是什么（这里实际上在运行环境中多是 K8s Token 的 Authenticator）
	  2.  然后调用 `sign` 函数来生成数字证书，同时返回自己的公钥给客户端
	  
	  忘记的同学可以看这里
	  
	  ![image](https://jsd.cdn.zzko.cn/gh/yanickxia/picx-images-hosting@master/20240319/image.7zq6sw145v.webp) ([View Highlight](https://read.readwise.io/read/01hv1y560gcbpk8bak173zxq2n))
	- ![mtls](https://jsd.cdn.zzko.cn/gh/yanickxia/picx-images-hosting@master/20240401/mtls.969iklao17.svg) ([View Highlight](https://read.readwise.io/read/01hv1y786shdhcy89sfj886815))