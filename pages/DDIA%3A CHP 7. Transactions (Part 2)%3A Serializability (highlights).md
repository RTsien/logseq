title:: DDIA: CHP 7. Transactions (Part 2): Serializability (highlights)
author:: [[Murat (noreply@blogger.com)]]
full-title:: "DDIA: CHP 7. Transactions (Part 2): Serializability"
category:: #articles
url:: http://muratbuffalo.blogspot.com/2024/10/ddia-chp-7-transactions-part-2.html
summary:: We are continuing from the first part of our Chapter 7 review.  Serializable isolation ensures that the final result of concurrent transactions is equivalent to if they had been run one at a time, without any concurrency. This eliminates any concurrency anomalies, since it ensures the transactions would behave as they would in a sequential environment.Databases offering serializability typically use one of three methods:Executing transactions literally in a serial order Two-phase locking (2PL)Optimistic concurrency control, like serializable snapshot isolation (SSI)For now, we will focus on single-node databases. The book discusses how these methods apply to distributed systems in Chapter 9.Actual Serial ExecutionSerial transaction execution is used in systems like VoltDB/H-Store, Redis, and Datomic. While limiting the database to a single thread can boost performance by eliminating coordination overhead, it restricts throughput to the capacity of one CPU core. Unlike traditional databases, these systems don't a...
![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEjUFkuNc9rd7IXhXwHtrU9cBz7vJsIXFymhmywXM2LoQ2iI2RRShORxXqyud7BVZOPvpZ56R72BzgOq65jMd0nP5K-I13qeDK3999sMh__FUQ4g1aL3RBEloXKI0qqiVMczm4BXDb7rFudjK-PdSSqDdCyCpT367tgdu6RGs3Uw4EbqyiAqZIAMV3QsNz0/s72-w400-h250-c/Screenshot%202024-10-08%20at%2010.41.49%E2%80%AFAM.png)

- Highlights first synced by [[Readwise]] [[Oct 14th, 2024]]
	- Serial transaction execution is used in systems like VoltDB/H-Store, Redis, and Datomic. While limiting the database to a single thread can boost performance by eliminating coordination overhead, it restricts throughput to the capacity of one CPU core. Unlike traditional databases, these systems don't allow interactive multi-statement transactions. The entire transaction logic must be pre-submitted as a stored procedure (see Figure 7-9). With all data in memory, this allows the procedure to run quickly without waiting on network or disk I/O. ([View Highlight](https://read.readwise.io/read/01ja2c60m4m08363416s5n7cjk))
	- For decades, 2PL was the primary method for enforcing serializability. Here, several transactions can read the same object using a shared lock, but any write operation requires exclusive access:
	  
	  •   If Transaction A reads an object and Transaction B wants to write to it, B must wait until A commits or aborts.
	  •   If Transaction A writes to an object and Transaction B wants to read it, B must wait until A finishes.
	  •   A transaction must hold onto its locks until it commits or aborts --hence the "two-phase" name: acquiring locks while executing, releasing them at the end.
	  
	  In 2PL, writers block readers and vice versa. Contrast this with snapshot isolation, whose mantra is **"readers never block writers, and writers never block readers"**. ([View Highlight](https://read.readwise.io/read/01ja2f6dp8gyrv4hbrmk5hmcaz))