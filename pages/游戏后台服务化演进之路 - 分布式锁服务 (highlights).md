title:: 游戏后台服务化演进之路 - 分布式锁服务 (highlights)
author:: [[woa.com]]
full-title:: "游戏后台服务化演进之路 - 分布式锁服务"
category:: #articles
url:: https://km.woa.com/articles/show/531810?ts=1638975890#%E6%AD%BB%E9%94%81
![](https://readwise-assets.s3.amazonaws.com/static/images/article1.be68295a7e40.png)

- Highlights first synced by [[Readwise]] [[Dec 25th, 2023]]
	- 防死锁机制：对多把锁按照key大小排序后发起锁请求过程。这样，当两个独立的事务并发地请求完全相同的多把锁时，总是会以相同的key顺序开始请求。前一个key未获得锁时，不会触发后续key的锁请求，也就不会发生“死锁”了。 ([View Highlight](https://read.readwise.io/read/01hjg0cqd7e12bpy88x92wfrtf))