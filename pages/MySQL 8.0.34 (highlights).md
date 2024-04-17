title:: MySQL 8.0.34 (highlights)
author:: [[Peter Alvaro & Kyle Kingsbury]]
full-title:: "MySQL 8.0.34"
category:: #articles
url:: https://jepsen.io/analyses/mysql-8.0.34
summary:: MySQL 8.0.34's Read Uncommitted, Read Committed, and Serializable isolation levels appear to satisfy PL-1, PL-2, and PL-3, respectively. However, MySQL's "Repeatable Read" does not satisfy PL-2.99 Repeatable Read and exhibits G2-item anomalies including write skew. The transactions in MySQL's "Repeatable Read" violate internal consistency and do not meet the definitions of ANSI SQL standards. This behavior rules out several isolation levels, including Read Atomic, Causal, Consistent View, Prefix, and Parallel snapshot isolation. This issue has not been resolved in MySQL 8.0.34.
![](https://readwise-assets.s3.amazonaws.com/static/images/article2.74d541386bbf.png)

- Highlights first synced by [[Readwise]] [[Apr 13th, 2024]]
	- In order to discuss the nuances of SQL isolation levels, we must first explain some history. In 1977 Gray, Lorie, Putzolu, and Traiger published [Granularity of Locks and Degrees of Consistency in a Shared Data Base](https://www.cs.cmu.edu/~natassa/courses/15-721/papers/GrayLocks.pdf), which introduced four increasingly safe degrees of transaction consistency. In 1973 IBM developed System R, one of the first relational databases, and shortly thereafter [introduced SQL](https://learnsql.com/blog/history-of-sql/) as a query language for it. System R’s success spawned a slew of relational databases using SQL, many with distinct flavors of concurrency control. Starting in [1986](https://archive.org/details/federalinformati127nati) ANSI [released](https://blog.ansi.org/sql-standard-iso-iec-9075-2023-ansi-x3-135/)[1](https://jepsen.io/analyses/mysql-8.0.34#fn1) a [series of standards](https://learnsql.com/blog/history-of-sql-standards/) codifying SQL behavior. The third revision of the standard, SQL-92, defined the semantics of concurrent transactions through four transaction isolation levels, again with increasing degrees of safety. As with Gray et al., these isolation levels were related to the behavior of increasingly conservative locking regimes. However, to allow databases which used non-locking concurrency control, ANSI phrased their levels in terms of three possible phenomena which should not occur. ([View Highlight](https://read.readwise.io/read/01hv9kke0n07nj3ahdet77xedw))
	- P1 (“Dirty Read”)
	  
	  SQL-transaction T1 modifies a row. SQL-transaction T2 then reads that row before T1 performs a COMMIT. If T1 then performs a ROLLBACK, T2 will have read a row that was never committed and that may thus be considered to have never existed.
	  
	  P2 (“Non-Repeatable Read”)
	  
	  SQL-transaction T1 reads a row. SQL-transaction T2 then modifies or deletes that row and performs a COMMIT. If T1 then attempts to reread the row, it may receive the modified value or discover that the row has been deleted.
	  
	  P3 (“Phantom”)
	  
	  SQL-transaction T1 reads the set of rows N that satisfy some <search condition>. SQL-transaction T2 then executes SQL-statements that generate one or more rows that satisfy the <search condition> used by SQL-transaction T1. If SQL-transaction T1 then repeats the initial read with the same <search condition>, it obtains a different collection of rows. ([View Highlight](https://read.readwise.io/read/01hv9kn9k4601gbtyf93yazqev))
	- In 1995 Berenson, Bernstein, Gray,[3](https://jepsen.io/analyses/mysql-8.0.34#fn3) Melton, and the O’Neils published [A Critique of ANSI SQL Isolation Levels](https://www.microsoft.com/en-us/research/wp-content/uploads/2016/02/tr-95-51.pdf), which laid out critical flaws in these definitions. “The three ANSI phenomena are ambiguous. Even their broadest interpretations do not exclude anomalous behavior.” ([View Highlight](https://read.readwise.io/read/01hv9knnh1tx7rgnd92e9kzs1n))
	- In 1999, [Atul Adya built on Berenson et al.’s critique](https://pmg.csail.mit.edu/papers/adya-phd.pdf) and developed formal and implementation-independent definitions of various transaction isolation levels, including those in ANSI SQL.[4](https://jepsen.io/analyses/mysql-8.0.34#fn4) As he notes:
	  
	  > The ANSI definitions are imprecise because they allow at least two interpretations; furthermore, the anomaly interpretation is definitely incorrect. The preventative interpretation [meaning Berenson et al.’s interpretation which added P0, expanded P3, and so on] is correct in the sense that it rules out undesirable (i.e., non-serializable) histories. However, this interpretation is overly restrictive since it also rules out correct behavior that does not lead to inconsistencies and can occur in a real system. Thus, any system that allows such histories is disallowed by this interpretation, e.g., databases based on optimistic mechanisms.
	  
	  Adya first defines a dependency graph between transactions. There are three main types of dependencies, which we summarize informally:
	  
	  Write-Write
	  
	  Transaction *T*1 writes some version *x*1 of object *x*, which transaction *T*2 overwrites by installing the next version of *x*: *x*2.
	  
	  Write-Read
	  
	  Transaction *T*1 writes version *x*1, which transaction *T*2 reads.
	  
	  Read-Write
	  
	  Transaction *T*1 reads version *x*1, which transaction *T*2 overwrites by installing the next version of *x*: *x*2.
	  
	  Adya then defines portable isolation levels PL-1, PL-2, PL-2.99, and PL-3, which capture what the ANSI SQL standard (arguably) intended. Each level rules out progressively broader kinds of cycles in the transaction dependency graph:
	  
	  PL-1 (“Read Uncommitted”)
	  
	  Prohibits G0 (“write cycle”): a cycle of write-write dependencies. This is analogous to Berenson’s P0 (“dirty write”).
	  
	  PL-2 (“Read Committed”)
	  
	  Prohibits G0 and G1. G1 consists of three anomalies: G1a (“aborted read”), G1b (“intermediate read”)[5](https://jepsen.io/analyses/mysql-8.0.34#fn5), and G1c (“cyclic information flow”): a cycle of write-write or write-read dependencies. This captures the essence of the preventative interpretation of P1.
	  
	  PL-2.99 (“Repeatable Read”)
	  
	  Prohibits G0, G1, and G2-item: a cycle involving write-write, write-read, or read-write edges *without predicates*. This captures the essence of ANSI SQL Repeatable Read, which is distinguished from Serializable only by predicate safety.
	  
	  PL-3 (“Serializable”)
	  
	  Prohibits G0, G1, and G2: a cycle involving write-write, write-read, or read-write edges (with or without predicates). This guarantees equivalence to a serial execution.
	  
	  Adya’s dependency graph-based isolation levels resolved the ambiguities of the ANSI definitions, and remains the most widely-used formalism for characterizing transaction histories and anomalies. Jepsen generally uses Adya’s formalism.
	  
	  Although the database community has known for decades that ANSI SQL’s isolation level definitions are broken, the standard’s language remained unchanged. The same ambiguous, incomplete definitions are still present in the [2023 revision of the standard](https://webstore.ansi.org/standards/iso/isoiec90752023-2502169). ([View Highlight](https://read.readwise.io/read/01hv9ksa9n566t8gdgy2zqz0c5))
	- 1.3 MySQL Isolation
	  
	  The [transaction isolation levels](https://dev.mysql.com/doc/refman/8.0/en/innodb-transaction-isolation-levels.html) documentation for MySQL indicates that MySQL with InnoDB “offers all four transaction isolation levels described by the SQL:1992 standard”: [Read Uncommitted](https://dev.mysql.com/doc/refman/8.0/en/innodb-transaction-isolation-levels.html#isolevel_read-uncommitted), [Read Committed](https://dev.mysql.com/doc/refman/8.0/en/innodb-transaction-isolation-levels.html#isolevel_read-committed), [Repeatable Read](https://dev.mysql.com/doc/refman/8.0/en/innodb-transaction-isolation-levels.html#isolevel_repeatable-read), and [Serializable](https://dev.mysql.com/doc/refman/8.0/en/innodb-transaction-isolation-levels.html#isolevel_serializable). The documentation goes on to explain how MySQL achieves these isolation levels.
	  
	  At MySQL Read Uncommitted, transactions should behave “like [Read Committed](https://dev.mysql.com/doc/refman/8.0/en/innodb-transaction-isolation-levels.html#isolevel_read-uncommitted),” except for allowing [dirty read](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_dirty_read): an anomaly where a read observes “data that was updated by another transaction but not yet committed.”
	  
	  At MySQL [Read Committed](https://dev.mysql.com/doc/refman/8.0/en/innodb-transaction-isolation-levels.html#isolevel_read-committed), every individual [consistent read](https://dev.mysql.com/doc/refman/8.0/en/innodb-consistent-read.html) reads from a fresh snapshot of committed state. A “consistent read” is the default behavior for reads (e.g. `SELECT * FROM problems`) and is the focus of this report. [There are also](https://dev.mysql.com/doc/refman/8.0/en/innodb-locking-reads.html) stronger reads (e.g. `SELECT ... FOR UPDATE`) which explicitly request locks, and weaker reads (e.g. `SELECT ... SKIP LOCKED`) which skip some of the default locks.
	  
	  MySQL [Repeatable Read](https://dev.mysql.com/doc/refman/8.0/en/innodb-transaction-isolation-levels.html#isolevel_repeatable-read), the default isolation level, ensures safety through a snapshot mechanism:
	  
	  > Consistent reads within the same transaction read the snapshot established by the first read. This means that if you issue several plain (nonlocking) `SELECT` statements within the same transaction, these `SELECT` statements are consistent also with respect to each other.
	  
	  MySQL’s [consistent read documentation](https://dev.mysql.com/doc/refman/8.0/en/innodb-consistent-read.html) further emphasizes that reads should operate on a snapshot of the database taken by the first read in a transaction.
	  
	  > If the transaction isolation level is `REPEATABLE READ` (the default level), all consistent reads within the same transaction read the snapshot established by the first such read in that transaction….
	  > 
	  > Suppose that you are running in the default `REPEATABLE READ` isolation level. When you issue a consistent read (that is, an ordinary `SELECT` statement), InnoDB gives your transaction a timepoint according to which your query sees the database. If another transaction deletes a row and commits after your timepoint was assigned, you do not see the row as having been deleted. Inserts and updates are treated similarly.
	  
	  The documentation for [Serializable](https://dev.mysql.com/doc/refman/8.0/en/innodb-transaction-isolation-levels.html#isolevel_serializable) isolation says Serializable is “like `REPEATABLE READ`, but `InnoDB` implicitly converts all plain `SELECT` statements to `SELECT ... FOR SHARE` if `autocommit` is disabled.”
	  
	  There ends the isolation level documentation. However, if one digs deeper into the [consistent read documentation](https://dev.mysql.com/doc/refman/8.0/en/innodb-consistent-read.html), there is a curious note on the semantics of Repeatable Read:
	  
	  > The snapshot of the database state applies to `SELECT` statements within a transaction, not necessarily to DML statements. If you insert or modify some rows and then commit that transaction, a `DELETE` or `UPDATE` statement issued from another concurrent `REPEATABLE READ` transaction could affect those just-committed rows, even though the session could not query them. If a transaction does update or delete rows committed by a different transaction, those changes do become visible to the current transaction.
	  
	  This is confusing: the ANSI SQL standard and MySQL’s [own reference manual](https://dev.mysql.com/doc/refman/8.0/en/sql-data-manipulation-statements.html) both consider `SELECT` to be a DML statement, but this note seems to think they’re different. It appears that writes made by a Repeatable Read transaction can affect rows that the transaction could not read. But what does it mean for a different transaction’s updates to become visible to the current transaction? How does that align with MySQL’s claim that multiple reads in a Repeatable Read transaction “read the snapshot established by the first read”? What happened to the timepoint assigned by the first read? ([View Highlight](https://read.readwise.io/read/01hv9ktsgtvz7g2x44w42c4cbw))
	- Cabral and Murphy repeat that Repeatable Read “allows a transaction to see the same data for values it has already read regardless of whether or not the data has been changed.” In their section on multi-version concurrency control, they emphasize the independence of transaction snapshots:
	  
	  > If a second transaction starts, it “checks out” its own copy of the data. If the first transaction makes changes and commits, the second transaction will not see the data. The second transaction can only work with the data it has. There is no way to update the data that the second transaction sees, though the second transaction could issue a ROLLBACK and start the transaction again to see the new data.
	  
	  This is also wrong: writing a row modifies the transaction’s local copy of the data. ([View Highlight](https://read.readwise.io/read/01hv9mpk6th5cy4j4qvz2rj1c3))
		- 💡: 读是快照读，写可是直接写
	- However, the section on Serializable isolation actually demonstrates (perhaps inadvertently) that MySQL’s Repeatable Read allows both lost update, a change in read snapshot, and a resulting internal consistency violation! It then shows that `Serializable` prevents those anomalies. It doesn’t name the anomalies, instead opting to say that “this doesn’t make sense”, but the behavior is visible to a careful reader. It’s not clear if the authors realize the example contradicts their earlier claims about non-repeatable reads and snapshot integrity. ([View Highlight](https://read.readwise.io/read/01hv9mrqrnk4ke2q3gs00facv3))
	- The core problem is that MySQL claims to implement Repeatable Read but actually provides something much weaker. We see two avenues to resolve this problem.
	  
	  The first is to keep MySQL’s behavior as it is, and to clearly document the consistency model “Repeatable Read” actually provides. There is precedent in other databases: PostgreSQL’s Repeatable Read [is actually Snapshot Isolation](https://jepsen.io/analyses/postgresql-12.3), and exhibits behaviors which violate PL-2.99 Repeatable Read. However, PostgreSQL’s documentation eventually [mentions](https://www.postgresql.org/docs/current/transaction-iso.html#XACT-REPEATABLE-READ) that their Repeatable Read implementation is actually Snapshot Isolation. MySQL could similarly document that their “Repeatable Read” means “Read Committed, plus some sort of guarantees that hold until the transaction writes something, at which point mysteries occur.” A precise characterization of those mysteries would be most welcome.
	  
	  The second option is to treat these behaviors as bugs and fix them. Jepsen would be delighted if MySQL and other vendors were to commit to providing PL-2.99 Repeatable Read. However, even satisfying the incomplete, ambiguous ANSI definition of Repeatable Read would be an improvement over current affairs.
	  
	  In the meantime, MySQL users who require PL-2.99 or ANSI Repeatable Read should be cautious of MySQL Repeatable Read. Reads may not be repeatable, or even reflect a snapshot of committed state. The common ORM pattern in which a transaction reads an object into memory, modifies it, then writes it back within a transaction, may cause committed updates to be silently lost. Users requiring Repeatable Read semantics should use MySQL’s Serializable isolation instead. Alternatively, they can selectively strengthen reads performed at `READ COMMITTED` using locking techniques like `SELECT ... FOR UPDATE`. ([View Highlight](https://read.readwise.io/read/01hv9mtfqg0t39wxv6e73agggw))
	- MySQL’s binlog replication appears fragile. We observed a number of mysterious scenarios in which replication halted in our local Jepsen tests. We also found that a few minutes of testing could [completely break](https://mastodon.jepsen.io/@jepsen/111231274947177218) AWS RDS’s MySQL replication: even a simple `CREATE DATABASE` would succeed on the primary and fail to appear on the secondaries. We waited an hour without observing recovery. MySQL’s [default settings are known to be unsafe](https://dev.mysql.com/doc/refman/8.0/en/replication-solutions-unexpected-replica-halt.html) in replicated systems. We made no attempt to promote nodes from secondaries to primaries, or to explore [exciting topologies](https://mariadb.com/kb/en/replication-overview/#common-replication-setups) like ring or star replication. Future work might explore these behaviors. ([View Highlight](https://read.readwise.io/read/01hv9n3hkbq9rn7774egb1ngbm))
	- Twenty-eight years after Berenson et al. demonstrated that ANSI SQL’s isolation levels are ambiguous and incomplete, seven revisions of the ANSI & ISO standards have left its definitions unchanged.[16](https://jepsen.io/analyses/mysql-8.0.34#fn16) P0 is still legal at every level up to Repeatable Read. We still don’t know whether circular information flow is legal at Read Committed. P3 still doesn’t mention deletes. Internal behavior remains unspecified. [The](https://www.vldb.org/pvldb/vol7/p181-bailis.pdf) [research](https://www.cs.cornell.edu/lorenzo/papers/Crooks17Seeing.pdf) [community](https://arxiv.org/abs/1903.00731) [has](https://software.imdea.org/~andrea.cerone/works/Framework.pdf) [moved](https://software.imdea.org/~gotsman/papers/si-podc16.pdf) [on](http://www.cs.ox.ac.uk/people/hongseok.yang/paper/popl14-final.pdf) [to](https://asc.di.fct.unl.pt/~nmp/pubs/europar-2-2013.pdf) [new](https://dsf.berkeley.edu/cs286/papers/ssi-tods2005.pdf) [formalisms](https://www.inf.usi.ch/faculty/pedone/Paper/2004/IC_TECH_REPORT_200421.pdf). Many are based on [Adya’s 1999 thesis](https://pmg.csail.mit.edu/papers/adya-phd.pdf), which struggled to capture “what the SQL standard actually meant.” ([View Highlight](https://read.readwise.io/read/01hv9n66172x6y5km2dw8dp8wd))