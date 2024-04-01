title:: Transaction management：两阶段锁（two-phase locking） (highlights)
author:: [[知乎专栏]]
full-title:: "Transaction management：两阶段锁（two-phase locking）"
category:: #articles
url:: https://zhuanlan.zhihu.com/p/59535337
summary:: The text discusses the concept of two-phase locking (2PL) as a simple algorithm to ensure conflict serializability in transaction management. Two-phase locking involves using shared and exclusive locks on data objects to control the order of conflicting read and write operations in a transaction schedule. By following the two-phase locking protocol, transactions can guarantee data consistency and isolation while minimizing complexity in database systems.
![](https://readwise-assets.s3.amazonaws.com/media/uploaded_book_covers/profile_1160846/v2-8f3740a0103dce55d7d0683379cfb297_720w.jpg)

- Highlights first synced by [[Readwise]] [[Apr 1st, 2024]]
	- 开始之前我们先简单复习一下上一篇文章中相关的概念，如果读者有需要也可以返回阅读[上一篇文章](https://zhuanlan.zhihu.com/p/57579023)。
	  
	  •   **数据库模型**: 我们的讨论继续使用上一篇文章中的模型：数据库由不可再分（indivisible）的、互不重叠（non-overlapping）的数据对象（data objects）的集合构成：${o1,o2,…,on}\{o_1, o_2,\dots,o_n\}\{o_1, o_2,\dots,o_n\}$，每一个object都有一个取值范围（domain of values）。这个系统的一个*状态（state）* 就是一个从object到value的映射。数据库的操作有 $read (r(o_i) )$和$write (w(o_i) )$。
	  •   **事务（transaction）**: Transaction是对数据库系统中读写操作的更高一层抽象，代表了“一个单位”的数据库操作。一个transaction可能包含对多个数据对象的多个读写操作，但是这些操作被视为一个整体。
	  •   **调度（schedule）**: Schedule是多个transactions的交错，表示了多个transaction中的多个数据操作的执行顺序。
	  •   **可串行性（serializability）**: 一个可串行的（serializable）schedule “等价于”某个serial schedule，就是其中涉及的所有transaction的某种序列化执行。更具体地说，根据对“等价”的不同定义，我们得出了三种不同的serializability定义：**final state serializability (FSR)**，**view serializability (VSR)**，和**conflict serializability (CSR)**。其中，因为[[CSR]]确保了data consistency和transaction isolation并且实现复杂度最低，在实际数据库系统中我们一般选择实现CSR。 ([View Highlight](https://read.readwise.io/read/01htbqktg0vx0w4g1qmsrhfyae))
	- 在上一篇文章中我们分析conflict serializability的时候发现，造成inconsistent schedule的原因是多个transaction之间冲突的读写操作的执行顺序。通常来说在并发情景中，这种执行顺序差异造成的错误被称为race condition。比如在 $S: w_1(x)~w_2(x)~w_2(y)~w_1(y)$ 这个inconsistent schedule的例子中， $T_1$ 抢先写入了 $x$ ，但是 $T_2$ 抢先写入了 $y$ ，这样的race condition导致了 $S$ 中出现了conflict环，没法等价于任何一种串行的调度执行。
	  
	  那么我们在CS其他领域，比如多线程控制中，是怎么处理race condition的呢？没错，就是通过加锁来控制仅有一个线程能进入执行critical section。所以，一个很自然的想法就是也通过加锁的方法来控制schedule中的冲突读写操作。但是在[[CSR]]这个问题中，用锁控制的“critical section”并不是直观的一段代码，而是冲突操作的顺序。怎么通过加锁来保证所有冲突操作的顺序一致呢？这就有了最简单的两阶段锁([[2PL]])的概念。 ([View Highlight](https://read.readwise.io/read/01htbqscmhzsjqfyjcnw38td99)) #card
		- 💡: 2PL用来解决CSR的conflict环问题
		- 💡：conflict环是等价于死锁吗？好像不是，conflict 环只会导致事务的串行化被破坏（出现了因果交错），但执行不会被终止。而死锁是会导致执行终止。当然如果使用锁机制来保证串行化，如果锁机制实现有问题导致了死锁，这就会导致事务无法继续执行，这也就是[[2PL]]理论的价值：让开发以一种简单的锁机制实现可串行化。
	- [[2PL]]的定义：如果一个transaction释放了它所持有的**任意一个锁**，那它就**再也不能获取任何锁**。 ([View Highlight](https://read.readwise.io/read/01htbqq1pagemdyg9gjpy3p49r))  #card
		- two-phase locking名字的由来了：在2PL协议下，每个transaction都会经过两个阶段：在第一个阶段里，transaction根据需要不断地获取锁，叫做 ***growing phase (expanding phase)***；在第二个阶段里，transaction开始释放其持有的锁，根据2PL的规则，这个transaction不能再获得新的锁，所以它所持有的锁逐渐减少，叫做 ***shrinking phase (contracting phase)***。
		  id:: 84576927-f514-4119-b761-a32d0334068a
		- [[2PL]]的算法非常简单，但是它为什么能够确保transaction的执行满足CSR呢？它的正确性证明也非常简单精妙[UCI CS223]
			- > 我们用**反证法**证明：假设 $S$ 是一个使用2PL得出的schedule，并假设 $S$ 并不满足CSR。因为 $S$ 并不满足CSR，所以 $S$ 的serialization graph $SG(S)$ 中**必然存在一个环**（不明白的读者请参考[上一篇文章](https://zhuanlan.zhihu.com/p/57579023)），不失一般性地，我们把这个环记做 $T_i\to T_j\to\cdots\to T_i$。考虑环中的一条边 $T_i\to T_j$ ，这条边的存在说明 $T_i$ 中存在某个操作 $o_i$ ，它和 $T_j$ 中的某个操作 $o_j$ 冲突，并且 $o_i\prec_S o_j$ 。因为 $o_i, o_j$ 冲突，我们根据conflicting operations的定义可知， $o_i$ 和 $o_j$ 涉及同一个数据对象，并且其中至少有一个写操作。那么根据2PL的lock compatibility，这两个操作所需要的锁一定冲突。又因为 $o_i\prec_S o_j$ ，所以 $T_i$ 先取得了锁 $l$ ；并且在此之后 $T_j$ 取得了和 $l$ 冲突的锁（因为 $T_j$ 执行了 $o_j$ ），所以这时 $T_i$一定已经释放了 $l$ 。所以，我们可以总结得出： $T_i$ **一定在** $T_j$ **获得** $T_j$ **需要的所有锁之前释放了某个锁**，换句话说，$(T_i\to T_j)\in SG(S)\Rightarrow T_i$ **一定在** $T_j$ **完成expanding phase之前进入了shrinking phase**。依此类推， $(T_j\to T_k)\in SG(S)\Rightarrow T_j$ 一定在 $T_k$ 完成expanding phase之前进入了shrinking phase...... 所以，环 $(T_i\to T_j\to\cdots\to T_i)\in SG(S)\Rightarrow T_i$ 在它自己完成expanding phase之前就进入了shrinking phase，得出矛盾，假设不成立，所以使用2PL得出的schedule必然满足CSR。证毕。 ([View Highlight](https://read.readwise.io/read/01htbqqxh6v0vgrvpzc2ve838s))