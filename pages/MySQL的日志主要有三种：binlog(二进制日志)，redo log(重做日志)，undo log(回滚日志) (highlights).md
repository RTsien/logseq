title:: MySQL的日志主要有三种：binlog(二进制日志)，redo log(重做日志)，undo log(回滚日志) (highlights)
author:: [[wangqilong.net]]
full-title:: "MySQL的日志主要有三种：binlog(二进制日志)，redo log(重做日志)，undo log(回滚日志)"
category:: #articles
url:: https://www.wangqilong.net/2021/01/28/mysql-de-shi-wu-ri-zhi/#toc-heading-10
![](https://readwise-assets.s3.amazonaws.com/static/images/article1.be68295a7e40.png)

- Highlights first synced by [[Readwise]] [[Sep 10th, 2024]]
	- Statement模式
	  
	  每一条修改数据的 sql 都会记录到 master 的 binlog 中，slave 在复制的时候，sql 进程会解析成和原来在 master 端执行时的相同的 sql 再执行。
	  
	  •   **优点**：在 statement 模式下首先就是解决了 row 模式的缺点，不需要记录每一行数据的变化，从而减少了 binlog 的日志量，节省了 I/O 以及存储资源，提高性能。因为它只需要记录在 master 上执行的语句的细节以及执行语句的上下文信息。
	  •   **缺点**：在 statement 模式下，由于它是记录的执行语句，所以，为了让这些语句在 slave 端也能正确执行，那么它还必须记录每条语句在执行的时候的一些相关信息，即上下文信息，以保证所有语句在 slave 端和在 master 端执行结果相同。另外就是，由于 MySQL 现在发展比较快，很多新功能不断的加入，使 MySQL 的复制遇到了不小的挑战，自然复制的时候涉及到越复杂的内容，bug 也就越容易出现。在statement 中，目前已经发现不少情况会造成 MySQL 的复制出现问题，主要是在修改数据的时候使用了某些特定的函数或者功能才会出现，比如：sleep() 函数在有些版本中就不能被正确复制，在存储过程中使用了 last_insert_id() 函数，可能会使 slave 和 master 上得到不一致的 id 等等。由于 row 模式是基于每一行来记录变化的，所以不会出现类似的问题。 ([View Highlight](https://read.readwise.io/read/01j3f19r8hk65mm43jv9n96ez6))