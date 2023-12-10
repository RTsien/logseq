title:: Go 语言内存分配器的实现原理 (highlights)
author:: [[draveness.me]]
full-title:: "Go 语言内存分配器的实现原理"
category:: #articles
url:: https://draveness.me/golang/docs/part3-runtime/ch07-memory/golang-memory-allocator/
![](https://readwise-assets.s3.amazonaws.com/static/images/article3.5c705a01b476.png)

- Highlights first synced by [[Readwise]] [[Nov 26th, 2023]]
	- 线程缓存属于每一个独立的线程，它能够满足线程上绝大多数的内存分配需求，因为不涉及多线程，所以也不需要使用互斥锁来保护内存，这能够减少锁竞争带来的性能损耗。当线程缓存不能满足需求时，运行时会使用中心缓存作为补充解决小对象的内存分配，在遇到 32KB 以上的对象时，内存分配器会选择页堆直接分配大内存。 ([View Highlight](https://read.readwise.io/read/01hg3yxzjgdwx32grn4v1a94rd))
		- **Note**: 线程缓存隶属于线程，不涉及锁
		  中心缓存用于补充线程缓存不足的情况
		  页堆应对32kb以上大缓存对象
- New highlights added [[Dec 8th, 2023]] at 10:35 PM
	- 线性的堆内存需要预留大块的内存空间，但是申请大块的内存空间而不使用是不切实际的，不预留内存空间却会在特殊场景下造成程序崩溃。虽然连续内存的实现比较简单，但是这些问题也没有办法忽略。
	  
	  稀疏内存是 Go 语言在 1.11 中提出的方案，使用稀疏的内存布局不仅能移除堆大小的上限[5](https://draveness.me/golang/docs/part3-runtime/ch07-memory/golang-memory-allocator/#fn:5)，还能解决 C 和 Go 混合使用时的地址空间冲突问题[6](https://draveness.me/golang/docs/part3-runtime/ch07-memory/golang-memory-allocator/#fn:6)。不过因为基于稀疏内存的内存管理失去了内存的连续性这一假设，这也使内存管理变得更加复杂：
	  
	  ![](https://img.draveness.me/2020-02-29-15829868066468-heap-after-go-1-11.png) ([View Highlight](https://read.readwise.io/read/01hh47s2ep3sc7gkja5hhmnthp))
		- 💡: 其实就是把原本一整块的连续堆区，分成了多个连续64MB的小区。
	- 不同平台和架构的二维数组大小可能完全不同，如果我们的 Go 语言服务在 Linux 的 x86-64 架构上运行，二维数组的一维大小会是 1，而二维大小是 4,194,304，因为每一个指针占用 8 字节的内存空间，所以元信息的总大小为 32MB。由于每个 [`runtime.heapArena`](https://draveness.me/golang/tree/runtime.heapArena) 都会管理 64MB 的内存，整个堆区最多可以管理 256TB 的内存，这比之前的 512GB 多好几个数量级。 ([View Highlight](https://read.readwise.io/read/01hh485w9b919kha5kakam3z5e))
		- 💡: 4194304*64MB=256T
	- Go 语言的内存分配器包含内存管理单元、线程缓存、中心缓存和页堆几个重要组件，本节将介绍这几种最重要组件对应的数据结构 [`runtime.mspan`](https://draveness.me/golang/tree/runtime.mspan)、[`runtime.mcache`](https://draveness.me/golang/tree/runtime.mcache)、[`runtime.mcentral`](https://draveness.me/golang/tree/runtime.mcentral) 和 [`runtime.mheap`](https://draveness.me/golang/tree/runtime.mheap)，我们会详细介绍它们在内存分配器中的作用以及实现。
	  
	  ![](https://img.draveness.me/2020-02-29-15829868066479-go-memory-layout.png)
	  
	  **图 7-10 Go 程序的内存布局**
	  
	  所有的 Go 语言程序都会在启动时初始化如上图所示的内存布局，每一个处理器都会分配一个线程缓存 [`runtime.mcache`](https://draveness.me/golang/tree/runtime.mcache) 用于处理微对象和小对象的分配，它们会持有内存管理单元 [`runtime.mspan`](https://draveness.me/golang/tree/runtime.mspan)。 ([View Highlight](https://read.readwise.io/read/01hh4e861b5m1gm1yfrj55k729))
	- 每个 [`runtime.mspan`](https://draveness.me/golang/tree/runtime.mspan) 都管理 `npages` 个大小为 8KB 的页，这里的页不是操作系统中的内存页，它们是操作系统内存页的整数倍 ([View Highlight](https://read.readwise.io/read/01hh4eemczp4q5xtvqytqqhq7n))
	- [`runtime.mspan`](https://draveness.me/golang/tree/runtime.mspan) 会以两种不同的视角看待管理的内存，当结构体管理的内存不足时，运行时会以页为单位向堆申请内存：
	  
	  ![](https://img.draveness.me/2020-02-29-15829868066492-mspan-and-pages.png) ([View Highlight](https://read.readwise.io/read/01hh4ehf4fgsfkhztfc3hrjpbe))