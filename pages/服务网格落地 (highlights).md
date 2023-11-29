title:: 服务网格落地 (highlights)
author:: [[aliyun.com]]
full-title:: "服务网格落地"
category:: #articles
url:: https://help.aliyun.com/zh/mesh/product-overview/service-mesh-implementation
![](https://readwise-assets.s3.amazonaws.com/static/images/article0.00998d930354.png)

- Highlights first synced by [[Readwise]] [[Nov 29th, 2023]]
	- SOFAMosn 支持两种 IO 模型：
	  
	  •   **Golang 经典模型**：在蚂蚁集团内部的落地场景，连接数不是瓶颈，都在几千或者上万的量级，蚂蚁集团选择了 Golang 经典模型 goroutine-per-connection。
	    
	    ![golang经典模型.png](https://help-static-aliyun-doc.aliyuncs.com/assets/img/zh-CN/4561121161/p228548.png)
	    
	    **模型缺陷**：协程数量与连接数量成正比，大链接场景下，协程数量过多，存在以下开销：
	    
	    •   Stack 内存开销
	        
	    •   Read buffer 开销
	        
	    •   Runtime 调度开销
	        
	  •   **RawEpoll 模型**：也就是 Reactor 模式，即 I/O 多路复用（I/O multiplexing） + 非阻塞 I/O （non-blocking I/O）模式。对于接入层和网关有大量长连接的场景，更加适合于 RawEpoll 模型。
	    
	    ![RawEpoll模式2.png](https://help-static-aliyun-doc.aliyuncs.com/assets/img/zh-CN/4561121161/p228549.png)
	  
	  **步骤说明**：
	  
	  1.  建立连接：
	    
	    向 Epoll 注册 oneshot 可读事件监听。此时不允许有协程调用 `conn.read`，以免与 `runtime Netpoll` 冲突。
	    
	  2.  可读事件到达，从 goroutine pool 挑选一个协程进行读事件处理。由于使用的是 oneshot 模式，该 fd 后续可读事件不会再触发。
	    
	  3.  请求处理过程中，协程调度与经典 Netpoll 模式一致。
	    
	  4.  请求处理完成，将协程归还给协程池，同时将 fd 重新添加到 RawEpoll 中。 ([View Highlight](https://read.readwise.io/read/01hgb40tbh7khvqy691h9kh0w0))
		- 💡: trpc-go的server端也是goroutine-per-connection模型。只不过针对每一个请求的处理分为串行和并发异步两种。
	- **协程模型**
	  
	  ![协程模型.png](https://help-static-aliyun-doc.aliyuncs.com/assets/img/zh-CN/4561121161/p228550.png)
	  
	  **说明**
	  
	  •   一个 TCP 连接对应一个 Read 协程，执行收包和协议解析。
	    
	  •   一个请求对应一个 Worker 协程，执行业务处理、Proxy 和 Write 逻辑。
	    
	  •   在常规模型中，一个 TCP 连接有 Read/Write 两个协程，蚂蚁团队取消了单独的 Write 协程，让 workerpool 工作协程代替它，减少了调度延迟和内存占用。 ([View Highlight](https://read.readwise.io/read/01hgb41a98tfn7vccsmveqewq5))
	- **容器升级**：主要流程包括下述几个方面。
	  
	  ![容器升级](https://help-static-aliyun-doc.aliyuncs.com/assets/img/zh-CN/4561121161/p228553.png)
	  
	  1.  先注入一个新的 SOFAMosn。
	    
	  2.  通过共享卷的 UnixSocket 去检查是否存在老的 SOFAMosn。
	    
	  3.  如果存在老的 SOFAMosn，就和老的 SOFAMosn 进行连接迁移，然后老的 SOFAMosn 退出。 ([View Highlight](https://read.readwise.io/read/01hgb4v0p8wcjtt2q0z5pewcfp))
	- **SOFAMosn 的连接迁移**：连接迁移的核心是内核 Socket 的迁移和应用数据的迁移。连接不断，且对用户无感知。![mosn迁移](https://help-static-aliyun-doc.aliyuncs.com/assets/img/zh-CN/5561121161/p228554.png) ([View Highlight](https://read.readwise.io/read/01hgb4v4r269v60s2vy735bp8q))
	- **SOFAMosn 的 Metric 迁移**：蚂蚁团队使用了共享内存来共享新老进程的 Metric 数据，保证在迁移的过程中 Metric 数据也是正确的。![metrics迁移.png](https://help-static-aliyun-doc.aliyuncs.com/assets/img/zh-CN/5561121161/p228555.png) ([View Highlight](https://read.readwise.io/read/01hgb4th5w22t71jf94mh7yb0x))