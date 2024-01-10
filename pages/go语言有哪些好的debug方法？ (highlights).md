title:: go语言有哪些好的debug方法？ (highlights)
author:: [[zhihu.com]]
full-title:: "go语言有哪些好的debug方法？"
category:: #articles
url:: https://www.zhihu.com/question/40980436/answer/767289819
![](https://readwise-assets.s3.amazonaws.com/static/images/article3.5c705a01b476.png)

- Highlights first synced by [[Readwise]] [[Jan 10th, 2024]]
	- MemStat & GC
	  
	  首先先获得一些基本的信息，内存使用和GC情况。
	  
	  MemStats:
	  
	    // read mem stats
	    var m runtime.MemStats
	    runtime.ReadMemStats(&m)
	  
	  GC:
	  
	    // disable gc when start
	    GOGC=off go run main.go
	     
	     
	    // disable gc and manually trigger gc
	    debug.SetGCPercent(-1)
	    runtime.GC()
	     
	    // read gc stats
	    var g debug.GCStats
	    debug.ReadGCStats(&g)
	  
	  MemStats中我们关注的是Alloc/HeapAlloc，这个值代表了当前heap的大小，另外就是HeapObjects，代表了当前heap中有多少个对象，同时应该关注一下Frees，这个值代表了一共释放过多少的对象，可能当前内存使用已经降下来了但过去某个时间曾经升高过。
	  
	  同时可以结合GCStats查看一下GC的情况， LastGC代表了上次GC时间，NumGC代表了一共GC过多少次，PauseTotal总暂停时间以及Pause暂停历史。 ([View Highlight](https://read.readwise.io/read/01hkrk1b55dvw7f5j9kdwm5010))