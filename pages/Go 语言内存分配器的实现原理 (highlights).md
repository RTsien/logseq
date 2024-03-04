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
- New highlights added [[Feb 29th, 2024]] at 8:39 AM
	- 运行时会使用 [`runtime.mSpanStateBox`](https://draveness.me/golang/tree/runtime.mSpanStateBox) 存储内存管理单元的状态 [`runtime.mSpanState`](https://draveness.me/golang/tree/runtime.mSpanState)：
	  
	    type mspan struct {
	    	...
	    	state       mSpanStateBox
	    	...
	    }
	    
	  
	  该状态可能处于 `mSpanDead`、`mSpanInUse`、`mSpanManual` 和 `mSpanFree` 四种情况。当 [`runtime.mspan`](https://draveness.me/golang/tree/runtime.mspan) 在空闲堆中，它会处于 `mSpanFree` 状态；当 [`runtime.mspan`](https://draveness.me/golang/tree/runtime.mspan) 已经被分配时，它会处于 `mSpanInUse`、`mSpanManual` 状态，运行时会遵循下面的规则转换该状态：
	  
	  •   在垃圾回收的任意阶段，可能从 `mSpanFree` 转换到 `mSpanInUse` 和 `mSpanManual`；
	  •   在垃圾回收的清除阶段，可能从 `mSpanInUse` 和 `mSpanManual` 转换到 `mSpanFree`；
	  •   在垃圾回收的标记阶段，不能从 `mSpanInUse` 和 `mSpanManual` 转换到 `mSpanFree`；
	  
	  设置 [`runtime.mspan`](https://draveness.me/golang/tree/runtime.mspan) 状态的操作必须是原子性的以避免垃圾回收造成的线程竞争问题。 ([View Highlight](https://read.readwise.io/read/01hq4tgsrszr5zgtf0r92k6tqk))
	- Go 语言的内存管理模块中一共包含 67 种跨度类，每一个跨度类都会存储特定大小的对象并且包含特定数量的页数以及对象，所有的数据都会被预选计算好并存储在 [`runtime.class_to_size`](https://draveness.me/golang/tree/runtime.class_to_size) 和 [`runtime.class_to_allocnpages`](https://draveness.me/golang/tree/runtime.class_to_allocnpages) 等变量中 ([View Highlight](https://read.readwise.io/read/01hq4thfyd1q4p1hcdz363agen))
	- 线程缓存会通过中心缓存的 [`runtime.mcentral.cacheSpan`](https://draveness.me/golang/tree/runtime.mcentral.cacheSpan) 方法获取新的内存管理单元 ([View Highlight](https://read.readwise.io/read/01hq4wdg77xdkryhv7kn53jh3e))
	- 页堆中包含一个长度为 136 的 [`runtime.mcentral`](https://draveness.me/golang/tree/runtime.mcentral) 数组，其中 68 个为跨度类需要 `scan` 的中心缓存，另外的 68 个是 `noscan` 的中心缓存：
	  
	  ![](https://img.draveness.me/2020-02-29-15829868066525-mheap-and-mcentrals.png) ([View Highlight](https://read.readwise.io/read/01hq5bjx7fqca9kz4hy8nhw36j))
	- 在除了 Windows 以外的 64 位操作系统中，每一个 [`runtime.heapArena`](https://draveness.me/golang/tree/runtime.heapArena) 都会管理 64MB 的内存空间 ([View Highlight](https://read.readwise.io/read/01hq5bm4d45kywkkmr2rp5snm2))
	- 为了阻止内存的大量占用和堆的增长，我们在分配对应页数的内存前需要先调用 [`runtime.mheap.reclaim`](https://draveness.me/golang/tree/runtime.mheap.reclaim) 方法回收一部分内存，随后运行时通过 [`runtime.mheap.allocSpan`](https://draveness.me/golang/tree/runtime.mheap.allocSpan) 分配新的内存管理单元，我们会将该方法的执行过程拆分成两个部分：
	  
	  1.  从堆上分配新的内存页和内存管理单元 [`runtime.mspan`](https://draveness.me/golang/tree/runtime.mspan)；
	  2.  初始化内存管理单元并将其加入 [`runtime.mheap`](https://draveness.me/golang/tree/runtime.mheap) 持有内存单元列表； ([View Highlight](https://read.readwise.io/read/01hq5bptv8th61xea4jg0h9ycx))
	- 使用 [`runtime.gomcache`](https://draveness.me/golang/tree/runtime.gomcache) 获取线程缓存并判断申请内存的类型是否为指针。我们从这个代码片段可以看出 [`runtime.mallocgc`](https://draveness.me/golang/tree/runtime.mallocgc) 会根据对象的大小执行不同的分配逻辑，在前面的章节也曾经介绍过运行时根据对象大小将它们分成微对象、小对象和大对象，这里会根据大小选择不同的分配逻辑：
	  
	  ![](https://img.draveness.me/2020-02-29-15829868066537-allocator-and-memory-size.png)
	  
	  **图 7-19 三种对象**
	  
	  •   微对象 `(0, 16B)` — 先使用微型分配器，再依次尝试线程缓存、中心缓存和堆分配内存；
	  •   小对象 `[16B, 32KB]` — 依次尝试使用线程缓存、中心缓存和堆分配内存；
	  •   大对象 `(32KB, +∞)` — 直接在堆上分配内存； ([View Highlight](https://read.readwise.io/read/01hq5cb6wp3hf7dxe186fszt96))