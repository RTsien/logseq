title:: Python进阶18垃圾回收GC (highlights)
author:: [[ebook-python-study.readthedocs.io]]
full-title:: "Python进阶18垃圾回收GC"
category:: #articles
url:: https://ebook-python-study.readthedocs.io/zh-cn/latest/python%E8%BF%9B%E9%98%B618%E5%9E%83%E5%9C%BE%E5%9B%9E%E6%94%B6GC.html
summary:: Python uses reference counting as the main garbage collection mechanism, with additional support from mark and sweep and generational collection. The garbage collection in Python involves reference counting, mark and sweep, and generational collection to efficiently manage memory. The three main garbage collection methods in Python are reference counting, mark and sweep, and generational collection.
![](https://readwise-assets.s3.amazonaws.com/static/images/article3.5c705a01b476.png)

- Highlights first synced by [[Readwise]] [[Apr 11th, 2024]]
	- python采用的是**引用计数机制为主，标记-清除和分代收集两种机制为辅的策略**。
	  
	  Python GC主要使用**引用计数（reference counting）来跟踪和回收垃圾**。在引用计数的基础上，**通过“标记-清除”（mark and sweep）解决容器对象可能产生的循环引用问题**，**通过“分代回收”（generation collection）以空间换时间的方法提高垃圾回收效率。** ([View Highlight](https://read.readwise.io/read/01hv5xn3fm02r5xbpc98q91rhk))