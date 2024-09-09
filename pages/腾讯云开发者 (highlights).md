title:: 腾讯云开发者 (highlights)
author:: [[tencent.com]]
full-title:: "腾讯云开发者"
category:: #articles
url:: https://cloud.tencent.com/developer/article/2211842
summary:: The text discusses how appending to slices in Go can lead to data races when multiple goroutines operate on the same slice. It provides examples of both thread-safe and thread-unsafe operations, highlighting that the capacity of the slice affects the likelihood of data races. A solution is suggested: create a new slice to avoid conflicts instead of modifying a shared slice concurrently.
![](https://cloudcache.tencentcs.com/open_proj/proj_qcloud_v2/gateway/shareicons/cloud.png)

- Highlights first synced by [[Readwise]] [[Sep 10th, 2024]]
	- ***根据golang中slice的******数据结构******可知，slice依托数组实现，在底层数组容量充足时，append操作不是只读操作，会将元素直接加入数组的空闲位置。因此，在多协程 对全局slice进行append操作时，会操作同一个底层数据，导致读写冲突*** ([View Highlight](https://read.readwise.io/read/01j5ds83cxkgr2cna1tgfx7p98))
	- 如果原slice的容量小于1024，则新slie的容量将扩大为原来的2倍 2 、如果原slice的容量大于或等于1024，则新slice的容量将扩大为原来的1.25倍 在该规则的基础上，还会考虑元素类型与内存分配规则，对实际扩张值做一些微调。从这个规则中可以看出Go对slice的性能和空间使用率的思考。 1、当切片较小时，采用较大的扩容倍速，可以避免频繁地扩容，从而减少内存分配的2、次数和数据拷贝的代价当切片较大时，采用较小的扩容倍速，主要是为了避免浪费空间。 ([View Highlight](https://read.readwise.io/read/01j5dsct50v5t6cyz7fxfngq8c))