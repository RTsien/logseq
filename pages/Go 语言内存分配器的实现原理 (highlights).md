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