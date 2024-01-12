title:: 一文搞懂Go1.20内存分配器 (highlights)
author:: [[woa.com]]
full-title:: "一文搞懂Go1.20内存分配器"
category:: #articles
url:: https://km.woa.com/articles/show/572647
![](https://readwise-assets.s3.amazonaws.com/static/images/article3.5c705a01b476.png)

- Highlights first synced by [[Readwise]] [[Jan 12th, 2024]]
	- TCMalloc的核心原理是：**把内存分为多级管理，从而降低锁的粒度。每个线程都会自行维护一个独立的内存池，进行内存分配时优先从该内存池中分配，当内存池不足时才会向全局内存池申请，以避免不同线程对全局内存池的频繁竞争**。 ([View Highlight](https://read.readwise.io/read/01hkxqyk9me5zh7p8djktbnnjg))