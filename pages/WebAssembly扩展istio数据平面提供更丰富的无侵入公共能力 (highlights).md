title:: WebAssembly扩展istio数据平面提供更丰富的无侵入公共能力 (highlights)
author:: [[woa.com]]
full-title:: "WebAssembly扩展istio数据平面提供更丰富的无侵入公共能力"
category:: #articles
url:: https://km.woa.com/articles/show/575776
![](https://readwise-assets.s3.amazonaws.com/static/images/article1.be68295a7e40.png)

- Highlights first synced by [[Readwise]] [[Dec 6th, 2023]]
	- wasm字节码在浏览器的运行，需要依赖于wasm虚拟机提供runtime支持，在wasm虚拟机中提供了诸多系统api，以便wasm可以使用v8引擎提供的api。因为wasm字节码运行于虚拟机中，所以wasm程序和他的运行环境之间有完美的隔离性，运行环境可以信任wasm程序，由于这种特性，一些开发者看到了wasm技术运用在服务端的可能性：
	  
	  > 在服务端代码中嵌入wasm虚拟机，给wasm字节码提供一个与主程序隔离的runtime，从而提供一种安全可靠的插件机制。
	  
	  这个设想的落地，需要各个服务端语言支持创建wasm虚拟机，于是wasmedge这个项目诞生了：
	  
	  [https://wasmedge.org/](https://wasmedge.org/)
	  
	  wasmedge这个项目一个核心能力就是：在我们的代码中run一个wasm虚拟机，例如下面的golang代码主要靠这几个步骤创建一个wasm虚拟机，并运行一个wasm字节码程序： ([View Highlight](https://read.readwise.io/read/01hgz0r5hercnp3s1tz4v1ezzk))
	- envoy作为一个云原生网关，它借鉴了linux网络栈中的netfilter，设计一个filter链，用户可以实现自己的filter模块并嵌入filter链。
	  
	  ![](https://km.woa.com/asset/8538a760f5f24c6195bf16b21941e3f7?height=2799&width=3652)
	  
	  借助envoy的架构图，我们可以这样理解envoy的filter链：在连接层、网络层、应用层三层，envoy提供了三组对应的filter：listener filters、network filters、http connection manager。我们可以通过编写代码实现自己的filter，也可以用envoy内置的filter。
	  
	  前文提到，要实现一个envoy插件，我们只需要按照协议提供一个wasm字节码程序就可以。由于envoy的复杂性，编写符合协议的wasm字节码本身就是一个不容易的事，所以社区也涌现了一些编写envoy插件的sdk，比如：teratelab提供的proxy-wasm-go-sdk。以这个sdk为例，我们来看下我们可以实现哪些方法（部分）让envoy主程序来调用：
	  
	  ![](https://km.woa.com/asset/654c1b55c49643f882df94fc16b6c412?height=1422&width=1852)
	  
	  这个图中，我截取了envoy定义的一组http协议相关方法，我们可以在wasm程序中实现这些方法来编写envoy插件，这些方法的作用是：
	  
	  1.  OnHttpRequestHeaders：envoy在http协议头解包时调用，并传递http header的相关信息，我们可以给方法返回Action标志标识请求是否应该继续传递。  
	    
	  2.  OnHttpRequestBody：envoy在http请求体解包时调用，传递body相关信息  
	    
	  3.  ····（更多可以看文档） ([View Highlight](https://read.readwise.io/read/01hgz0rzb3g5kf6gcerhw1fbxk))