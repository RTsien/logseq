title:: Mosn 平滑升级原理解析 (highlights)
author:: [[MOSN]]
full-title:: "Mosn 平滑升级原理解析"
category:: #articles
url:: https://mosn.io/docs/products/structure/smooth-upgrade/
summary:: 如何在升级 Sidecar（MOSN）的时候而不影响业务，对于存量的长连接如何迁移，本文将为你介绍 MOSN 的解决之道。
![](https://readwise-assets.s3.amazonaws.com/static/images/article0.00998d930354.png)

- Highlights first synced by [[Readwise]] [[Jan 31st, 2024]]
	- 为什么 Nginx 和 Envoy 不需要具备 MOSN 这样的连接无损迁移方案，主要还是跟业务场景相关，Nginx 和 Envoy 主要支持的是 HTTP1 和 HTTP2 协议，HTTP1使用 connection: Close，HTTP2 使用 Goaway Frame 都可以让 Client 端主动断链接，然后新建链接到新的 New process，但是针对 Dubbo、SOFA PRC 等常见的多路复用协议，它们是没有控制帧，Old process 的链接如果断了就会影响请求的。 ([View Highlight](https://read.readwise.io/read/01hnevg10rwpw02x6fe84022k4))
	- MOSN 为了满足自身业务场景，开发了长连接迁移方案，把这条链接迁移到 New process 上，整个过程对 Client 透明，不需要重新建链接，达到请求无损的平滑升级。 ([View Highlight](https://read.readwise.io/read/01hnevhgfkntznz0kwbhgbpxm6))
	- ![正常流程](https://mosn.io/docs/products/structure/smooth-upgrade/normal-process.png)
	  
	  1.  Client 发送请求 Request 到 MOSN
	  2.  MOSN 转发请求 Request 到 Server
	  3.  Server 回复响应 Response 到 MOSN
	  4.  MOSN 回复响应 Response 到 Client
	  
	  上图简单介绍了一个请求的正常流程，我们后面需要迁移的是 TCP1 链接，也就是 Client 到 MOSN 的连接，MOSN 到 Server 的链接 TCP2 不需要迁移，因为 MOSN 访问 Server 是根据 LoadBalance 选择，我们可以主动控制断链建链。 ([View Highlight](https://read.readwise.io/read/01hnevm0et0rees0b5zf3she7f))
		- 💡: 不会担心2断开的时候，让服务器感知到断链？对于RPC之类的问题不大？http也就是重连，好像没什么问题
- New highlights added [[Jan 31st, 2024]] at 2:54 PM
	- 平滑升级流程触发条件
	  
	  有两个方式可以触发平滑升级流程：
	  
	  1.  MOSN 对 SIGHUP 做了监听，发送 SIGHUP 信号给 MOSN 进程，通过 ForkExec 生成一个新的 MOSN 进程。
	  2.  直接重新启动一个新 MOSN 进程。
	  
	  为什么提供两种方式？最开始我们支持的是方法1，也就是 nginx 和 Envoy 使用的方式，这个在虚拟机或者容器内替换 MOSN 二级制来升级是可行的，但是我们的场景需要满足容器间的升级，所以需要新拉起一个容器，就需要重新启动一个新的 MOSN 进程来做平滑升级，所以后续又支持了方法2。容器间升级还需要 operator 的支持，本文不展开叙述。 ([View Highlight](https://read.readwise.io/read/01hnf1w3mgqcrnepthsptr6w78))
	- 首先，老的 MOSN 在启动最后阶段会启动一个协程运行 `ReconfigureHandler()` 函数监听一个 Domain Socket（`reconfig.sock`）, 该接口的作用是让新的 MOSN 来感知是否存在老的 MOSN。
	  
	    func ReconfigureHandler() {
	        l, err := net.Listen("unix", types.ReconfigureDomainSocket)
	    
	        for {
	            uc, err := ul.AcceptUnix()
	            _, err = uc.Write([]byte{0})
	            reconfigure(false)
	        }
	    }
	    
	  
	  触发平滑升级流程的两种方式最终都是启动一个新的 MOSN 进程，然后调用`GetInheritListeners()`，通过 `isReconfigure()` 函数来判断本机是否存在一个老的 MOSN（就是判断是否存在 `reconfig.sock` 监听），如果存在一个老的 MOSN，就进入迁移流程，反之就是正常的启动流程。 ([View Highlight](https://read.readwise.io/read/01hnf20hg6teqrhkx6scr4xxq2))
- New highlights added [[Jan 31st, 2024]] at 5:51 PM
	- 如果进入迁移流程，新的 MOSN 将监听一个新的 Domain Socket（`listen.sock`），用于老的 MOSN 传递 listen FD 到新的 MOSN。FD 的传递使用了sendMsg 和 recvMsg。在收到 listen FD 之后，调用 `net.FileListener()` 函数生产一个 Listener。此时，新老 MOSN 都同时拥有了相同的 Listen 套接字。
	  
	    // FileListener returns a copy of the network listener corresponding
	    // to the open file f.
	    // It is the caller's responsibility to close ln when finished.
	    // Closing ln does not affect f, and closing f does not affect ln.
	    func FileListener(f *os.File) (ln Listener, err error) {
	        ln, err = fileListener(f)
	        if err != nil {
	            err = &OpError{Op: "file", Net: "file+net", Source: nil, Addr: fileAddr(f.Name()), Err: err}
	        }
	        return
	    }
	    
	  
	  这里的迁移和 Nginx 还是有一些区别，Nginx 是 fork 的方式，子进程自动就继承了 listen FD，MOSN 是新启动的进程，不存在父子关系，所以需要通过 sendMsg 的方式来传递。 ([View Highlight](https://read.readwise.io/read/01hnfcvh9jhm3gc9ggxpjm5ybk))
	- 在进入迁移流程和 Listen 的迁移过程中，一共使用了两个 Domain Socket：
	  
	  •   `reconfig.sock` 是 Old MOSN 监听，用于 New MOSN 来判断是否存在
	  •   `listen.sock` 是 New MOSN 监听，用于 Old MOSN 传递 listen FD
	  
	  两个 sock 其实是可以复用的，也可以用 `reconfig.sock` 进行 listen 的传递，由于一些历史原因搞了两个，后续可以优化为一个，让代码更精简易读。
	  
	  这儿再看看 Old MOSN 的处理，在收到 New MOSN 的通知之后，将进入`reconfigure(false)` 流程，首先就是调用 `sendInheritListeners()` 传递 listen FD，原因上面内容已经描述，最后调用 `WaitConnectionsDone()` 进入存量长链接的迁移流程。
	  
	    // 保留了核心流程
	    func reconfigure(start bool) {
	        if start {
	            startNewMosn()
	            return
	        }
	        // transfer listen fd
	        if notify, err = sendInheritListeners(); err != nil {
	            return
	        }
	        // Wait for all connections to be finished
	        WaitConnectionsDone(GracefulTimeout)
	    
	        os.Exit(0)
	    }
	    
	  
	  在 Listen FD 迁移之后，New MOSN 通过配置启动，然后在最后启动一个协程运行`TransferServer()`，将监听一个新的 `DomainSocket（conn.sock）`，用于后续接收 Old MOSN 的长连接迁移。迁移的函数是 `transferHandler()`
	  
	    func TransferServer(handler types.ConnectionHandler) {
	        l, err := net.Listen("unix", types.TransferConnDomainSocket)
	    
	        utils.GoWithRecover(func() {
	            for {
	                c, err := l.Accept()
	                go transferHandler(c, handler, &transferMap)
	    
	            }
	        }, nil)
	    }
	    
	  
	  Old MOSN 将通过 `transferRead()` 和 `transferWrite()` 进入最后的长链接迁移流程，下面主要分析这块内容。 ([View Highlight](https://read.readwise.io/read/01hnfcx3xcxr6naxq0m5c397rw))
- New highlights added [[Feb 4th, 2024]] at 11:22 AM
	- 长连接迁移流程[](https://mosn.io/docs/products/structure/smooth-upgrade#长连接迁移流程)
	  
	  ![长连接迁移过程](https://mosn.io/docs/products/structure/smooth-upgrade/long-connection-migrating-process.png)
	  
	  首先先粗略看一下新请求的迁移流程。
	  
	  1.  Client 发送请求到 MOSN
	  2.  MOSN 通过 domain socket(conn.sock) 把 TCP1 的 FD 和连接的状态数据发送给 New MOSN
	  3.  New MOSN 接受 FD 和请求数据创建新的 Conection 结构，然后把 Connection id 传给 MOSN，New MOSN 此时就拥有了 TCP1 的一个拷贝。
	  4.  New MOSN 通过 LB 选取一个新的 Server，建立 TCP3 连接，转发请求到 Server
	  5.  Server 回复响应到 New MOSN
	  6.  New MOSN 通过 MOSN 传递来的 TCP1 的拷贝，回复响应到 Client ([View Highlight](https://read.readwise.io/read/01hnhb4yd9k7ftyfjvp30aj7he))
	- 最后的残留响应迁移流程可能不太好理解，为什么不等所有响应完成之后才开始迁移，就不需要这个流程了？是因为在多路复用协议场景下，请求一直在发送，你不能总是找到一个时间点所有响应都完成了。 ([View Highlight](https://read.readwise.io/read/01hnhqrh94c44nhcsrsdk0kdmk))