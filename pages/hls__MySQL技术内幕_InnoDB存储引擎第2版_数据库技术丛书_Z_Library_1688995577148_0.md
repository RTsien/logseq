file:: [MySQL技术内幕_InnoDB存储引擎第2版_数据库技术丛书_Z_Library_1688995577148_0.pdf](../assets/MySQL技术内幕_InnoDB存储引擎第2版_数据库技术丛书_Z_Library_1688995577148_0.pdf)
file-path:: ../assets/MySQL技术内幕_InnoDB存储引擎第2版_数据库技术丛书_Z_Library_1688995577148_0.pdf

- 事务的实现 #innodb事务实现
  hl-page:: 554
  ls-type:: annotation
  id:: 64ce7d0d-fc5b-4bf2-af71-4ac323c3b166
  hl-color:: yellow
	- 事务隔离性由第6章讲述的锁来实现。原子性、一致性、持久性通过数据库的redo log和undo log来完成。redo log称为重做日志，用来保证事务的原子性和持久性。undo log用来保证事务的一致性。
		- 💡这个说法太过简单粗暴了