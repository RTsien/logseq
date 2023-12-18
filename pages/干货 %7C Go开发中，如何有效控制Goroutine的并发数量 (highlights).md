title:: 干货 | Go开发中，如何有效控制Goroutine的并发数量 (highlights)
author:: [[CSDN.net]]
full-title:: "干货 | Go开发中，如何有效控制Goroutine的并发数量"
category:: #articles
url:: https://blog.csdn.net/azl397985856/article/details/107171651
![](https://readwise-assets.s3.amazonaws.com/static/images/article1.be68295a7e40.png)

- Highlights first synced by [[Readwise]] [[Dec 14th, 2023]]
	- 在我们日常大部分场景下，不需要使用协程池。因为Goroutine非常轻量，默认2kb，使用go func()很难成为性能瓶颈。当然一些极端情况下需要追求性能，可以使用协程池实现资源的复用，例如FastHttp使用协程池性能提高许多。 ([View Highlight](https://read.readwise.io/read/01hhcmah36rmj5mx82hbanvh6h))