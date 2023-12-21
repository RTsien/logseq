title:: Envoy 源码解析 - 简书 (highlights)
author:: [[首页]]
full-title:: "Envoy 源码解析 - 简书"
category:: #articles
url:: https://www.jianshu.com/p/94f432658cf6
![](https://readwise-assets.s3.amazonaws.com/media/uploaded_book_covers/profile_1160846/8926363-c972a5e9946136c9.png)

- Highlights first synced by [[Readwise]] [[Dec 15th, 2023]]
	- envoy 如何利用多核？
	  
	  为了利用多核，envoy 的方案也是启动多个 event loop；而为了实现“单端口、利用多核”， envoy 的每个 event loop 线程会去 epoll_wait 同一个 socket。很神奇。
	  
	  Q: envoy 是怎么让多个worker 线程 epoll_wait 同一个 socket 的?  
	  A: 启动时，main thread 先 bind socket，然后通过复制 fd 或者 reuse_port 的方式让每个worker 都能监听相同port 的socket.  
	  
	  ![](https://upload-images.jianshu.io/upload_images/8926363-501161c462948332.png?imageMogr2/auto-orient/strip|imageView2/2/w/562/format/webp)
	  
	  image.png
	  
	  
	  [https://blog.mygraphql.com/zh/posts/low-tec/trace/trace-istio/trace-istio-part2/#listener](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.mygraphql.com%2Fzh%2Fposts%2Flow-tec%2Ftrace%2Ftrace-istio%2Ftrace-istio-part2%2F%23listener)
	  
	  Q: 复制 fd 是什么机制?  
	  A: 详见 [https://blog.mygraphql.com/zh/posts/low-tec/trace/trace-istio/trace-istio-part2/#](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.mygraphql.com%2Fzh%2Fposts%2Flow-tec%2Ftrace%2Ftrace-istio%2Ftrace-istio-part2%2F%23)原理  
	  
	  ![](https://upload-images.jianshu.io/upload_images/8926363-83c006530c4fed79.png?imageMogr2/auto-orient/strip|imageView2/2/w/995/format/webp)
	  
	  image.png
	  
	  Q: 什么是SO_REUSEPORT ?  
	  A: 简单来说，就是多个 server socket 监听相同的端口。每个 server socket 对应一个监听线程。内核 TCP 栈接收到客户端建立连接请求(SYN)时，按 TCP 4 元组(srcIP,srcPort,destIP,destPort) hash 算法，选择一个监听线程，唤醒之。新连接绑定到被唤醒的线程。所以相对于非 SO_REUSEPORT， 连接更为平均地分布到线程中（hash 算法不是绝对平均）。
	  
	  ![](https://upload-images.jianshu.io/upload_images/8926363-e43e192e0210f3f8.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)
	  
	  image.png
	  
	  
	  详见 [https://blog.mygraphql.com/zh/posts/cloud/istio/istio-tunning/istio-thread-balance/](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.mygraphql.com%2Fzh%2Fposts%2Fcloud%2Fistio%2Fistio-tunning%2Fistio-thread-balance%2F)
	  
	  Q: 怎么保证多个 worker线程的负载均衡？  
	  A: 设计目标是“每个线程绑定的连接数尽量一样”，如果 A、B有同样数量的连接数，但是 A 的连接很忙，B的连接很闲，那没办法。  
	  默认情况下，envoy 让内核做负载均衡。  
	  但问题是，kernel 只能做“同一时间来的请求均匀分给不同的线程”，而在长连接场景，假如 A 先接收一个长连接后，过了几分钟有新的连接，我们希望新连接交给别的线程、不要交给 A。也就是说，需要根据“当前状态”做LB，而 kernel 不会帮忙管这些状态。  
	  为了解决这个问题，Envoy 有种特殊配置 （ConnectionBalanceConfig），在用户态保证连接的负载均衡。  
	  大致原理是：一个 listener 线程 accept到连接后，进行负载均衡计算，如果算出来“我的负载太高，这个连接应该交给 B线程"，会转发给 B 线程的 listener ([View Highlight](https://read.readwise.io/read/01hhpa6ayhgxrt5wnqjh6r4w7f))