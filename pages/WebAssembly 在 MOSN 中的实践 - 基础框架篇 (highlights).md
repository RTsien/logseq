title:: WebAssembly 在 MOSN 中的实践 - 基础框架篇 (highlights)
author:: [[MOSN]]
full-title:: "WebAssembly 在 MOSN 中的实践 - 基础框架篇"
category:: #articles
url:: https://mosn.io/blog/posts/mosn-wasm-framework/
![](https://readwise-assets.s3.amazonaws.com/static/images/article3.5c705a01b476.png)

- Highlights first synced by [[Readwise]] [[Dec 11th, 2023]]
	- 当前 WebAssembly 技术仍处于发展阶段，Go 语言自身对 WebAssenbly 生态的支持仍有巨大的提升空间。我们在实践的过程中，也总是面临 Go 语言在 Wasm 生态中不够给力的情况。由于 Go 官方编译器还不支持将 Go 源码程序编译成 WASI 系统接口 (GOOS=wasi) 的 .wasm 文件，我们不得不借助 TinyGo 来完成 Go 扩展程序的编译，而这也导致我们需要面对 TinyGo 在语言特性支持程度、性能、稳定性等方面不足的痛点。与之相比，C++/Rust 对 Wasm 生态的支持程度就要完善得多。 ([View Highlight](https://read.readwise.io/read/01hhcac8x26p6x84kw1gncg18s))
- New highlights added [[Dec 11th, 2023]] at 7:55 PM
	- 规范的实现需要宿主侧和 Wasm 侧两边配合才能正常工作。对于 Wasm 侧，社区已经有 C++、Rust 和 Go 三种语言实现的 SDK，用户可以直接使用这些 SDK 来编写与宿主无关的 Wasm 扩展程序。而对于宿主侧，社区只提供了 C++ 和 Rust 的宿主侧实现。为此，我们在项目中使用 Go 语言对 Proxy-Wasm 规范的宿主侧进行了实现，并将其贡献给开源社区，使之成为社区推荐的 [Go-Host](https://github.com/mosn/proxy-wasm-go-host) 实现。需要强调的是，宿主侧实现并不依赖具体的网络代理程序，理论上任何直接通过 Host 程序与 Wasm 扩展进行交互。 ([View Highlight](https://read.readwise.io/read/01hhcaw9rnrbm9sh54h31gfs8r))
	- 我们以 HTTP 场景为例，介绍在 MOSN 中是如何通过 Proxy-Wasm 规范来与 Wasm 扩展程序进行交互，处理 HTTP 请求的。
	  
	  1.  MOSN 收到 HTTP 请求时，将请求解码成 Header、Body、Trailer 三元组结构，按照配置依次执行 StreamFilters
	  2.  执行到 Wasm StreamFilter 时，MOSN 将请求三元组传递给 Proxy-Wasm 宿主侧实现 proxy-wasm-go-host
	  3.  宿主侧 go-host 将 MOSN 请求三元组编码成规范指定的格式，并调用规范中的 proxy_on_request_headers 等接口，将请求信息传递至 Wasm 侧
	  4.  Wasm 侧 SDK 将请求数据从规范格式转换为便于用户使用的格式，随后调用用户编写的扩展代码
	  5.  用户代码返回，Wasm 侧将返回结果按规范格式传递回 MOSN 侧
	  6.  MOSN 继续执行后续 StreamFilter ([View Highlight](https://read.readwise.io/read/01hhcawjs10806hhs1vet2td2a))