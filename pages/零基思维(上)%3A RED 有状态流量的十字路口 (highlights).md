title:: 零基思维(上): RED 有状态流量的十字路口 (highlights)
author:: [[woa.com]]
full-title:: "零基思维(上): RED 有状态流量的十字路口"
category:: #articles
url:: https://iwiki.woa.com/p/4010260063
![](https://readwise-assets.s3.amazonaws.com/static/images/article3.5c705a01b476.png)

- Highlights first synced by [[Readwise]] [[Apr 11th, 2024]]
	- ![](https://iwiki.woa.com/tencent/api/attachments/s3/url?attachmentid=19232710) ([View Highlight](https://read.readwise.io/read/01hv3rjgegt859vt76xk87mbz6))
- New highlights added [[Apr 12th, 2024]] at 3:31 PM
	- Wasm Server 与 Wasm VM 关于高可用做了几个保障：
	  
	  •   Wasm VM 每秒钟以 HTTP 请求从 Wasm Server中拉取一次最新的差异化有状态服务各个实例的最新版本数据，并回写到 Wasm VM 内存中；
	    
	  •   Wasm Server 是无状态的，且天然支持 HPA，同时配置存储在内存中，并监听 k8s ShardedService CRDs 资源变更进行 PluginConfig 的内存配置更新，保障高可用。 ([View Highlight](https://read.readwise.io/read/01hv8eba6te847cgnrszf6gtsp))