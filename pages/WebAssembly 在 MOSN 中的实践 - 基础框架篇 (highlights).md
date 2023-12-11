title:: WebAssembly 在 MOSN 中的实践 - 基础框架篇 (highlights)
author:: [[MOSN]]
full-title:: "WebAssembly 在 MOSN 中的实践 - 基础框架篇"
category:: #articles
url:: https://mosn.io/blog/posts/mosn-wasm-framework/
![](https://readwise-assets.s3.amazonaws.com/static/images/article3.5c705a01b476.png)

- Highlights first synced by [[Readwise]] [[Dec 11th, 2023]]
	- 当前 WebAssembly 技术仍处于发展阶段，Go 语言自身对 WebAssenbly 生态的支持仍有巨大的提升空间。我们在实践的过程中，也总是面临 Go 语言在 Wasm 生态中不够给力的情况。由于 Go 官方编译器还不支持将 Go 源码程序编译成 WASI 系统接口 (GOOS=wasi) 的 .wasm 文件，我们不得不借助 TinyGo 来完成 Go 扩展程序的编译，而这也导致我们需要面对 TinyGo 在语言特性支持程度、性能、稳定性等方面不足的痛点。与之相比，C++/Rust 对 Wasm 生态的支持程度就要完善得多。 ([View Highlight](https://read.readwise.io/read/01hhcac8x26p6x84kw1gncg18s))