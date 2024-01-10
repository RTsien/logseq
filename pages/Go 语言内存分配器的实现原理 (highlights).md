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
- New highlights added [[Dec 8th, 2023]] at 4:11 PM
	- 编程语言的内存分配器一般包含两种分配方法，一种是线性分配器（Sequential Allocator，Bump Allocator），另一种是空闲链表分配器（Free-List Allocator），这两种分配方法有着不同的实现机制和特性，本节会依次介绍它们的分配过程。 ([View Highlight](https://read.readwise.io/read/01hh462br1aaewg75wt02t412s))
	- 线性分配（Bump Allocator）是一种高效的内存分配方法，但是有较大的局限性。当我们使用线性分配器时，只需要在内存中维护一个指向内存特定位置的指针，如果用户程序向分配器申请内存，分配器只需要检查剩余的空闲内存、返回分配的内存区域并修改指针在内存中的位置，即移动下图中的指针：
	  
	  ![](https://img.draveness.me/2020-02-29-15829868066435-bump-allocator.png) ([View Highlight](https://read.readwise.io/read/01hh4644wwgnc5grszcf6ahw4x))
		- 💡: 线性分配器无法重用已经被释放的内存空间。如果想利用，就需要配合一些空间回收的策略，但是这会导致历史指针失效。C和C++这种直接暴露指针的语言就无法使用线性分配器。
	- 空闲链表分配器（Free-List Allocator）可以重用已经被释放的内存，它在内部会维护一个类似链表的数据结构。当用户程序申请内存时，空闲链表分配器会依次遍历空闲的内存块，找到足够大的内存，然后申请新的资源并修改链表：
	  
	  ![](https://img.draveness.me/2020-02-29-15829868066446-free-list-allocator.png) ([View Highlight](https://read.readwise.io/read/01hh46amdydath26ws5pbh81yb))
		- 💡: 为了优化空闲内存块链表遍历耗时，一般会根据空闲内存块的大小分成多个链表——隔离适应（Segregated-Fit）。
	- 线程缓存分配（Thread-Caching Malloc，TCMalloc）是用于分配内存的机制，它比 glibc 中的 `malloc` 还要快很多[2](https://draveness.me/golang/docs/part3-runtime/ch07-memory/golang-memory-allocator/#fn:2)。Go 语言的内存分配器就借鉴了 TCMalloc 的设计实现高速的内存分配，它的核心理念是使用多级缓存将对象根据大小分类，并按照类别实施不同的分配策略。 ([View Highlight](https://read.readwise.io/read/01hh46zxrgkaw3ayb0kqz4ng6k))
	- ![](https://img.draveness.me/2020-02-29-15829868066457-multi-level-cache.png)
	  
	  **图 7-6 多级缓存内存分配**
	  
	  线程缓存属于每一个独立的线程，它能够满足线程上绝大多数的内存分配需求，因为不涉及多线程，所以也不需要使用互斥锁来保护内存，这能够减少锁竞争带来的性能损耗。当线程缓存不能满足需求时，运行时会使用中心缓存作为补充解决小对象的内存分配，在遇到 32KB 以上的对象时，内存分配器会选择页堆直接分配大内存。 ([View Highlight](https://read.readwise.io/read/01hh47425qdy4xsbwpqk3gq5hx))
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
- New highlights added [[Jan 5th, 2024]] at 5:13 PM
	- 在 amd64 的 Linux 操作系统上，[`runtime.mheap`](https://draveness.me/golang/tree/runtime.mheap) 会持有 4,194,304 [`runtime.heapArena`](https://draveness.me/golang/tree/runtime.heapArena)，每个 [`runtime.heapArena`](https://draveness.me/golang/tree/runtime.heapArena) 都会管理 64MB 的内存，单个 Go 语言程序的内存上限也就是 256TB。 ([View Highlight](https://read.readwise.io/read/01hkcc8v9rehph6af0h58mvzzd))
	- [`runtime.mcache`](https://draveness.me/golang/tree/runtime.mcache) 是 Go 语言中的线程缓存，它会与线程上的处理器一一绑定，主要用来缓存用户程序申请的微小对象。每一个线程缓存都持有 68 * 2 个 [`runtime.mspan`](https://draveness.me/golang/tree/runtime.mspan)，这些内存管理单元都存储在结构体的 `alloc` 字段中：
	  
	  ![](https://img.draveness.me/2020-02-29-15829868066512-mcache-and-mspans.png) ([View Highlight](https://read.readwise.io/read/01hkccwz7d7bxqzct62m1645sn))
	- [`runtime.mcache.refill`](https://draveness.me/golang/tree/runtime.mcache.refill) 会为线程缓存获取一个指定跨度类的内存管理单元，被替换的单元不能包含空闲的内存空间，而获取的单元中需要至少包含一个空闲对象用于分配内存： ([View Highlight](https://read.readwise.io/read/01hkcdd831g1c7afn08tth0xa3))