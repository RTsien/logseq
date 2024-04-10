title:: eBPF 是实现可观测性的关键技术 (highlights)
author:: [[让观测更简单]]
full-title:: "eBPF 是实现可观测性的关键技术"
category:: #articles
url:: https://mp.weixin.qq.com/s/Xhrz3Ne81nVhwuWAOS157Q
summary:: eBPF is a key technology for observability in cloud-native environments, offering advantages over traditional APM solutions. It provides zero-impact observability, helping DevOps and SRE teams with efficient troubleshooting and system optimization. DeepFlow, based on eBPF, offers features like full-stack metrics, distributed tracing, and continuous profiling for cloud-native applications.
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/x1ES4Jic395JzPvcG4ZKeaJ0Zn2sG14dVZApMaxibcZpNPbnnM2pUPia1ht5NJ0flWdV5wxibnlBVsvpmKnpb24frg/0?wx_fmt=jpeg)

- Highlights first synced by [[Readwise]] [[Apr 10th, 2024]]
	- *True application-level transparency, possibly our most challenging design goal, was achieved by restricting Dapper’s core tracing instrumentation to a small corpus of ubiquitous threading, control flow, and RPC library code.* ([View Highlight](https://read.readwise.io/read/01hv1sqvk30mvgbbssh9fd6hb3))
	- 零侵扰的分布式追踪（AutoTracing）是 DeepFlow 中的一个重大创新，在通过 eBPF 和 cBPF 采集调用日志时，DeepFlow 基于系统调用上下文计算出了 syscall_trace_id、thread_id、goroutine_id、cap_seq、tcp_seq 等信息，无需修改应用代码、无需注入 TraceID、SpanID 即可实现分布式追踪。
	  
	  目前 DeepFlow 除了跨线程（通过内存 Queue 或 Channel 传递信息）和异步调用以外，都能实现零侵扰的分布式追踪。此外也支持解析应用注入的唯一 Request ID（例如几乎所有网关都会注入 X-Request-ID）来解决跨线程和异步的问题。 ([View Highlight](https://read.readwise.io/read/01hv1sy7xdxxb9rans80b1bb4h))
	- ![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/x1ES4Jic395JzPvcG4ZKeaJ0Zn2sG14dVIcC6kicyzW2iapRle3VmjrynSYW5KrjSRZEgibhT5FK779tH3GibVx60Yw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
	  
	  微服务接入的各种网关，来自 theburningmonk@twitter ([View Highlight](https://read.readwise.io/read/01hv1t4v5gfremr5hn78wr77kc))