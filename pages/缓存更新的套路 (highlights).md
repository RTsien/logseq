title:: 缓存更新的套路 (highlights)
author:: [[陈皓]]
full-title:: "缓存更新的套路"
category:: #articles
url:: https://coolshell.cn/articles/17416.html
summary:: 看到好些人在写更新缓存数据代码时，先删除缓存，然后再更新数据库，而后续的操作会把数据再装载的缓存中。然而，这个是逻辑是错误的。试想，两个并发操作，一个是更新操作，另一个是查询操作，更新操作删除缓存后，
![](https://coolshell.cn/wp-content/uploads/2016/07/cache.png)

- Highlights first synced by [[Readwise]] [[Mar 12th, 2024]]
	- 更新缓存的的Design Pattern有四种：Cache aside, Read through, Write through, Write behind caching ([View Highlight](https://read.readwise.io/read/01hrry1hvs1qzzp18x7dzddf1n))
	- Read/Write Through Pattern
	  
	  我们可以看到，在上面的Cache Aside套路中，我们的应用代码需要维护两个数据存储，一个是缓存（Cache），一个是数据库（Repository）。所以，应用程序比较啰嗦。而Read/Write Through套路是把更新数据库（Repository）的操作由缓存自己代理了，所以，对于应用层来说，就简单很多了。**可以理解为，应用认为后端就是一个单一的存储，而存储自己维护自己的Cache。** ([View Highlight](https://read.readwise.io/read/01hrry63jj71cbck6y4xsaqn98))
	- Read Through
	  
	  Read Through 套路就是在查询操作中更新缓存，也就是说，当缓存失效的时候（过期或LRU换出），Cache Aside是由调用方负责把数据加载入缓存，而Read Through则用缓存服务自己来加载，从而对应用方是透明的。 ([View Highlight](https://read.readwise.io/read/01hrry5x5wvkvec72f8ah8s46s))
	- Write Through
	  
	  Write Through 套路和Read Through相仿，不过是在更新数据时发生。当有数据更新的时候，如果没有命中缓存，直接更新数据库，然后返回。如果命中了缓存，则更新缓存，然后再由Cache自己更新数据库（这是一个同步操作）
	  
	  下图自来Wikipedia的[Cache词条](https://en.wikipedia.org/wiki/Cache_(computing))。其中的Memory你可以理解为就是我们例子里的数据库。
	  
	  ![Write-through_with_no-write-allocation](https://coolshell.cn/wp-content/uploads/2016/07/460px-Write-through_with_no-write-allocation.svg_.png) ([View Highlight](https://read.readwise.io/read/01hrry1zy5knq9kztgdxdfreyv))