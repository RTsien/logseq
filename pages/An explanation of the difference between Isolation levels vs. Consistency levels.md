- http://dbmsmusings.blogspot.com/2019/08/an-explanation-of-difference-between.html
- #数据库 #隔离级别 #一致性级别 #[[Daniel Abadi]] #分布式学习资料
-
- [[数据库隔离性]]
  	数据库允许事务执行起来就像没有其他并行运行的事务一样，即使实际上可能存在大量并行运行的事务，这样就不会读写到其他并发事务产生的临时的、不完整的、中止的或者其他不正确的数据。
  	perfect isolation will ensure that the code remains correct even when there is other code running concurrently in the system that may read or write the same data. Thus, in order to achieve perfect isolation, the system must ensure that when transactions are running concurrently, the final state is equivalent to a state of the system that would exist if they were run serially.
  		**how and when writes become visible** 
  	扩展阅读
  		[[Introduction to Transaction Isolation Levels]]
- [[数据库一致性]]
  	不同的情境下有不同的定义。但是当一个现代系统提供多个一致性级别时，它们是从数据库的客户端视图来定义一致性的
  	Perfect consistency (in this post, we are going to call linearizability “perfect” even though strict consistency has slightly stronger guarantees) ensures both: that every thread of execution observes the same write ordering AND that write ordering matches reality (if write A completes before write B starts, no thread will see the write to B happening before the write to A). This guarantee is also true for reads: if a write to A completes before a read of A begins, then the write of A will be visible to the subsequent read of A (assuming A was not overwritten by a different write request in the interim).
  		 how and when writes become visible
  	扩展阅读
  		[[Overview of Consistency Levels in Database Systems]]
  [[The basic problem is that historically, as we described above, consistency levels are only designed for single-operation actions]]. 
  	consistency levels最初是在哪篇文章被提出的，当时是否的确是这样