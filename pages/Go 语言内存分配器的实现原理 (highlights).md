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