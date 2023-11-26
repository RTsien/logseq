title:: 推荐：七周七并发模型 (highlights)
author:: [[liaoxuefeng.com]]
full-title:: "推荐：七周七并发模型"
category:: #articles
url:: http://www.liaoxuefeng.com/article/0014299733752043b02cf5aefa642e889e9d3181b1dd92d000
![](https://www.liaoxuefeng.com/files/attachments/1013529600075008/l)

- Highlights first synced by [[Readwise]] [[Nov 26th, 2023]]
	- 无论什么并发模型，归根到底，设计思想就是：
	  
	  1.  数据要不变
	    
	  2.  数据要不变
	    
	  3.  数据要不变
	    
	  
	  在这个指导思想下实现并发，比传统的多线程+锁容易多了。任何数据，只要保持不变特性，就可以反复执行，因为结果是一样的，这样分布式和容错的实现就简单多了。
	  
	  你可能会问，数据怎么可能不变呢？其实不变特性是指数据在一个时间点上是不变的，想想版本控制系统就明白了。
	  
	  如果设计数据库模型时，实现了不变数据结构，那么将大大简化数据库事务的代码，并且极大地提升系统性能。遗憾的是，大多数开发人员设计的数据库模型都复杂得完全无法实现并发。 ([View Highlight](https://read.readwise.io/read/01hg3s27as1a019g108me7qh08))
		- **Note**: CAS