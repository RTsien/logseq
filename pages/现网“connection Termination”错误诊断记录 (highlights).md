title:: 现网“connection Termination”错误诊断记录 (highlights)
author:: [[woa.com]]
full-title:: "现网“connection Termination”错误诊断记录"
category:: #articles
url:: https://iwiki.woa.com/p/4013378220
summary:: The text discusses issues with connection termination errors in a network, specifically related to gRPC server settings. It identifies that the gRPC server's default TCP_USER_TIMEOUT of 20 seconds may lead to premature connection closures, causing clients to encounter errors. Suggested solutions include increasing the keepalive timeout on the gRPC server and enabling inbound traffic on the istio-proxy.
![](https://readwise-assets.s3.amazonaws.com/static/images/article0.00998d930354.png)

- Highlights first synced by [[Readwise]] [[Feb 11th, 2025]]
	- **client** → **istio-proxy** ---- **maybe cvm 中间件** ----> **istio-proxy** → **server**
	  
	  但因为我们 **server** 端是没有开 **istio-proxy inbound** 拦截的，所以实际上应该是这样：
	  
	  **client** → **istio-proxy** ---- **maybe cvm 中间件** ----> **server** ([View Highlight](https://read.readwise.io/read/01jkhjvz0e0qq0pa06vg7aa5sa))
		- 💡: red的inbound没有过envoy代理
	- **envoy** 目前 **server** 和 **client** 默认没有启用 **http2 keepalive**。而 **client envoy** 侧有个机制，从 **envoy** 文档来看，当下游连接处于 **idle** 时，最多 **1h** 会主动断开连接，所以也不用等到 server 端主动 2h 断开连接 ([View Highlight](https://read.readwise.io/read/01jkhjywfn2rbabmjzjtz26chr))