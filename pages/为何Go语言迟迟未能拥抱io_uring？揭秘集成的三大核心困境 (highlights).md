title:: 为何Go语言迟迟未能拥抱io_uring？揭秘集成的三大核心困境 (highlights)
author:: [[白明的赞赏账户]]
full-title:: "为何Go语言迟迟未能拥抱io_uring？揭秘集成的三大核心困境"
category:: #articles
url:: https://mp.weixin.qq.com/s/naDyE51tlkmXy58o1uKVFg
summary:: io_uring 是 Linux 的高性能异步 I/O。  
Go 未将其纳入标准库，因与运行时哲学冲突、容器安全禁用以及接口频繁变动。  
因此 Go 团队更偏向稳妥与兼容，社区对用户态实现的热情已减退。
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/cH6WzfQ94mag9eWcm3wIq55250dw1gVsQQ7TV7AeyP9Va70muBicrgtiaCqhWSZI6zZaCwmtTmqJyLicmRZib7GDtg/0?wx_fmt=jpeg)

- Highlights first synced by [[Readwise]] [[Aug 16th, 2025]]
	- 讨论伊始，Go 社区对 `io_uring` 寄予厚望，期待它能一举解决 Go 在 I/O 领域的两大历史痛点：
	  
	  1.  **真正的异步文件 I/O：** Go 的网络 I/O 基于 `epoll` 实现了非阻塞，但文件 I/O 本质上是阻塞的。为了避免阻塞系统线程，Go 运行时不得不维护一个线程池来处理文件操作。正如社区所期待的，`io_uring` 最大的吸引力在于**“移除对文件 I/O 线程池的需求”**，让文件 I/O 也能享受与网络 I/O 同等的高效与优雅。
	  2.  **极致的网络性能：** 对于高并发服务器，`io_uring` 通过将多个 `read`/`write` 操作打包成一次系统调用，能显著降低内核态与用户态切换的开销，这在“熔断”和“幽灵”漏洞导致 syscall 成本飙升的后时代尤为重要。 ([View Highlight](https://read.readwise.io/read/01k2qca1z2153z3vbwztrnd9h9))
	- `io_uring` 的性能优势源于**批处理**。但 Go 的标准库 API，如 `net.Conn.Read()`，是一个独立的、阻塞式的调用。Go 用户习惯于在独立的 goroutine 中处理独立的连接。如何将这些分散的独立 I/O 请求，在用户无感知的情况下，**“透明地”**收集起来，打包成批？这几乎是一个无解的难题。
	  
	  社区也提出了“每个 P (Processor) 一个 `io_uring` 环”的设想，但 Ian 指出这会引入极高的复杂性，包括环的争用、空闲 P 的等待与唤醒、P 与 M 切换时的状态管理等。正如一些社区成员所总结的，`io_uring` 需要一种全新的 I/O 模式，而这与 Go 现有网络模型的模式完全不同。强行“透明”集成，无异于“在不破坏现有 API 的情况下进行不必要的破坏”。 ([View Highlight](https://read.readwise.io/read/01k2qccezmnm4a2bt3h1azgh87))
		- 💡: io_uring的优势源于批处理，非常深刻直白的结论。
		  
		  golang的io读写对于goroutine来说是独立的，想基于批处理来实现看起来独立的表现，想起来就有复杂度，而且肯定会有不少反直觉的表现
	- 在 2024 年初，社区成员 `jakebailey` 抛出了一个重磅消息：**出于安全考虑，Docker 默认的 seccomp 配置文件已经禁用了 `io_uring`**。
	  
	  > **引用自 Docker 的 commit 信息:** "安全专家普遍认为 `io_uring` 是不安全的。事实上，Google ChromeOS 和 Android 已经关闭了它，所有 Google 生产服务器也关闭了它。"
	  
	  这个消息对标准库集成而言几乎是致命一击。Go 程序最常见的部署环境就是容器。一个不被“普遍情况”支持的特性，无论其性能多么优越，都难以成为Go运行时和标准库的基石。 ([View Highlight](https://read.readwise.io/read/01k2qcjve2sa8ra2qene2z6smy))
	- 核心困境三：追赶一个“移动的目标”
	  
	  在这场长达五年的讨论中，`io_uring` 自身也在飞速进化。其作者Jens Axboe 甚至亲自下场，解答了 Go 团队早期的疑虑，例如移除了并发数限制、解决了事件丢失问题等。
	  
	  但这恰恰揭示了第三重困境：**要集成一个仍在高速演进、API 不断变化的底层接口，本身就充满了风险和不确定性**。标准库追求的是极致的稳定性和向后兼容性。过早地依赖一个“移动的目标”，可能会带来持续的维护负担和潜在的破坏性变更。对于一个需要支持多个内核版本的语言运行时来说，这种复杂性是难以承受的。 ([View Highlight](https://read.readwise.io/read/01k2qcmt4xwe4ejgjnc03cvfj2))
	- `io_uring` 未能在 Go中落地，并非因为 Go 团队忽视性能，而是其成熟与审慎的体现。三大核心困境层层递进，揭示了其迟迟未能拥抱 `io_uring` 的深层原因：**哲学上的范式冲突、现实中的安全红线、以及工程上的稳定性质疑。** ([View Highlight](https://read.readwise.io/read/01k2qcnkgzxfw815wvmrz26wbj))
	- 与之形成鲜明对比的是，Rust 的 `tokio-uring` 库依然保持着旺盛的生命力，社区活跃，迭代频繁。这似乎在暗示，问题不仅在于 `io_uring` 本身，更在于它与特定语言运行时模型的“契合度”。[Go 运行时的 G-P-M 调度模型](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzIyNzM0MDk0Mg==&action=getalbum&album_id=4036682086282166273#wechat_redirect)和它所倡导的编程范式，使得社区自发的集成尝试也步履维艰，最终热情退潮。 ([View Highlight](https://read.readwise.io/read/01k2qcppz3btr5svcew1n1w76p))