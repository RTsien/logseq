- [[kafka]] [[零拷贝]] [[dma]] [[sendfile]] [[mmap]]
- [src](https://www.cnblogs.com/hongdada/p/16926179.html)
- ### mmap与sendFile区别
	- mmap 用于文件共享，很少用于socket操作，sendfile用于发送文件.
	- mmap 适合小数据量读写，sendFile 适合大文件传输。
		- {{embed ((64ace2b1-3cb7-4d1b-a7b4-91ce9fd5408a))}}
	- mmap 需要 4 次上下文切换，3 次数据拷贝；sendFile 需要 2 次上下文切换，最少 2 次数据拷贝。
	- sendFile 可以利用 DMA 方式，减少 CPU 拷贝，mmap 则不能（必须从内核拷贝到 Socket 缓冲区）。
		- ((64acdd69-e351-4cc5-abd0-ba9d30964952))
- ### mmap和共享内存的区别：
	- mmap是共享一个文件，共享内存是共享一段内存。mmap还可以写回到file.
- ### mmap缺点：
	- mmap 每次读入都是1页即4k，所以少于4k会造成大量内存碎片. 但是通过read,write也是这样的。
	- mmap适用场景，是取代read,write 文件.
- ### 使用mmap+write方式
	- 优点：即使频繁调用，使用小文件块传输，效率也很高
	  缺点：不能很好的利用DMA方式，会比sendfile多消耗CPU资源，内存安全性控制复杂，需要避免JVM Crash问题
- ### 使用[[sendfile]]方式
	- 优点：可以利用DMA方式，消耗CPU资源少，大块文件传输效率高，无内存安全新问题
	  缺点：**小块文件效率低于mmap方式，只能是BIO方式传输，不能使用NIO**
		- #TODO #🌟 有必要再认真求证一下，这是不是对的。为什么sendfile小文件效率就会比mmap低？
		  id:: 64ace2b1-3cb7-4d1b-a7b4-91ce9fd5408a
- rocketMQ 在消费消息时，使用了 mmap，因为小块数据传输比sendFile好。kafka 使用了 sendFile。
	- 这里应该是个谬误，rocketmq使用mmap的真实原因是从磁盘读取数据后需要在用户态修改数据再发送出去 
	  > sendFile相当于是原汁原味地读写，直接将硬盘上的文件送给网卡，但这种方式并不一定对所有场景都适用，比如如果你需要从硬盘上读取文件，然后经过一定修改之后再送给网卡的情况下，就不适合用sendFile。对于RocketMQ来说，因为RocketMQ将所有队列的数据都写入了CommitLog，消费者批量消费时需要读出来进行应用层过滤，所以就不能利用到sendfile+DMA的零拷贝方式，而只能用mmap。 [src](https://segmentfault.com/q/1010000041019461)