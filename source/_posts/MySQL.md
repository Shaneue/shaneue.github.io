---
title: MySQL
date: 2018-11-16 21:55:11
updated: 2019-08-12 12:00:00
tags: [Database]
typora-root-url: ../
---

The world's most popular open source database

<!-- more -->

## Normalization

数据库范式是为了数据库设计的规范化而提出的一些概念。从第一范式到第六范式要求越严格，约束性越高。通俗来讲，即范式越小，数据冗余越大（数据重复），表现为存储变大，且对更新数据不友好；而范式越大，查询需要跨表。最好的实践是，需要根据业务设计合理的数据库Schema，通常围绕在第三范式。

### 第三范式

关系模式中不允许有传递函数依赖。直观上判断，就是其它字段值只受限于主键。实践中，需要根据具体读写场景设计数据冗余度。

## Index

### 索引原理

- InnoDB与MyISAM索引区别

  ![](/images/mysql1.png)

现在MySQL默认使用InnoDB，而MyISAM不支持事务，正常情况不会使用。因此后面内容只针对InnoDB。

InnoDB中有BTREE（实际上是B+树，叶子结点存数据），HASH与FULLTEXT。

在InnoDB中，哈希索引是自适应的，用户不可干预。

### 聚簇索引

是依照主键来构建的。主键会默认构建索引。

在用普通索引检索时，会先根据该key找到主键，再通过主键找到记录，因此有两步操作。

## Concurrency

锁是加在索引上的。比如对记录进行更新，会先锁住主键索引，如果更新的是索引字段，又会再锁住该索引。另一种情况，如果更新时是根据普通索引字段检索，会先锁住该索引，再锁住主键索引。

InnoDB在RR隔离下不会出现幻读（严格来说还是有可能出现），因为使用了Next-key Lock。

### MVCC（Multi-Version Concurrency Control）

无锁方案实现的读写并发控制。

读操作分为当前读与快照读。

RR隔离下快照读可以防止不可重复读。

通过对记录添加两个隐藏列实现，一个是db_trx_id指向修改该记录的事务id，另一个是db_roll_ptr指向undo log中的未提交事务链表，可以找到修改的历史版本。

一致性读指一个事务根据快照来返回结果。比如一个数据在查询时被其他事务修改了，仍然会通过undo log来找回原来的数据生成读的结果。

MVCC可作用在RR、RC隔离下。

### 2PL（Two-Phase Locking）

在对数据进行读写操作的时候，首先获取数据的锁。

事务会一次性释放所有锁，并且不再获取其他锁。

事务会分成两个阶段：加锁与解锁，并且两个阶段不相交。

如果将两阶段锁实现成一次性获取所有锁，就不会存在死锁了。

### 死锁

InnoDB采用了上述两种方式实现读与写、写与写间的并发。因为仍然会采用悲观锁的机制，且并没有在事务中一次性获取所有锁，所以存在死锁风险。

### RC、RR

两个隔离方式都有较多使用场景。在RC下，不存在Gap锁，并发高、死锁少，但是在当前读时，会存在不可重复读与幻读的问题。RR当前读时，会采用Next-Key Lock来避免幻读。

官方解读：**如果需要尽可能地减少死锁发生，可以采用RC隔离。**

RC避免不了不可重复读。

### 常见的锁

#### 行锁

##### Gap Lock

间隙锁不可作用于RC隔离下

##### Next-Key Lock可以防止幻读

检索唯一索引不存在的值时，也会产生Gap锁。如存在唯一值，则降级为Record Lock

非唯一索引非范围检索时也加Gap锁，是因为索引值是跟主键值一起存放的，锁是加在索引节点上的

##### Record Lock

#### 其他锁

##### 共享锁（S）、排它锁（X）

##### 意向锁

##### 隐式锁

在只有一个事务操作数据行时，不需要加锁，只需要在隐藏列中标记一下即可

### 加锁过程示例

普通的select采用的是快照读，不加锁

select … lock in share mode  加共享锁

select … for update 加排它锁

复杂查询的加锁过程比较复杂，列出一些查看事务运行情况的命令

`show engine innodb status\G`

`show processlist`

`SELECT * FROM INFORMATION_SCHEMA.INNODB_LOCKS`

`SELECT * FROM INFORMATION_SCHEMA.INNODB_LOCK_WAITS`

`show open tables`

## InnoDB

### InnoDB Architecture

![](/images/mysql2.png)

### Pages, Extents, Segments, and Tablespaces

> Each tablespace consists of database pages. Every tablespace in a MySQL instance has the same page size. By default, all tablespaces have a page size of 16KB; you can reduce the page size to 8KB or 4KB by specifying the innodb_page_size option when you create the MySQL instance. You can also increase the page size to 32KB or 64KB.
>
> The pages are grouped into extents of size 1MB for pages up to 16KB in size (64 consecutive 16KB pages, or 128 8KB pages, or 256 4KB pages). For a page size of 32KB, extent size is 2MB. For page size of 64KB, extent size is 4MB. The “files” inside a tablespace are called segments in InnoDB.

#### How Pages Relate to Table Rows

> The maximum row length is slightly less than half a database page for 4KB, 8KB, 16KB, and 32KB innodb_page_size settings. For example, the maximum row length is slightly less than 8KB for the default 16KB InnoDB page size. For 64KB pages, the maximum row length is slightly less than 16KB.
>
> If a row does not exceed the maximum row length, all of it is stored locally within the page. If a row exceeds the maximum row length, variable-length columns are chosen for external off-page storage until the row fits within the maximum row length limit. External off-page storage for variable-length columns differs by row format:
>
> - COMPACT and REDUNDANT Row Formats: When a variable-length column is chosen for external off-page storage, InnoDB stores the first 768 bytes locally in the row, and the rest externally into overflow pages. Each such column has its own list of overflow pages. The 768-byte prefix is accompanied by a 20-byte value that stores the true length of the column and points into the overflow list where the rest of the value is stored. 
> - DYNAMIC and COMPRESSED Row Formats: When a variable-length column is chosen for external off-page storage, InnoDB stores a 20-byte pointer locally in the row, and the rest externally into overflow pages.
>
> LONGBLOB and LONGTEXT columns must be less than 4GB, and the total row length, including BLOB and TEXT columns, must be less than 4GB.

### InnoDB Features

#### Doublewrite Buffer

> The doublewrite buffer is a storage area where InnoDB writes pages flushed from the buffer pool before writing the pages to their proper positions in the InnoDB data files. If there is an operating system, storage subsystem, or mysqld process crash in the middle of a page write, InnoDB can find a good copy of the page from the doublewrite buffer during crash recovery.
>
> Although data is written twice, the doublewrite buffer does not require twice as much I/O overhead or twice as many I/O operations. Data is written to the doublewrite buffer in a large sequential chunk, with a single fsync() call to the operating system (except in the case that innodb_flush_method is set to O_DIRECT_NO_FSYNC).
>
> Prior to MySQL 8.0.20, the doublewrite buffer storage area is located in the InnoDB system tablespace. As of MySQL 8.0.20, the doublewrite buffer storage area is located in doublewrite files.

主要的优势在于写page是离散的，而doublewrite buffer时顺序写，效率影响不大。

#### Change Buffer

> The change buffer is a special data structure that caches changes to secondary index pages when those pages are not in the buffer pool. The buffered changes, which may result from INSERT, UPDATE, or DELETE operations (DML), are merged later when the pages are loaded into the buffer pool by other read operations.
>
> Unlike clustered indexes, secondary indexes are usually nonunique, and inserts into secondary indexes happen in a relatively random order. Similarly, deletes and updates may affect secondary index pages that are not adjacently located in an index tree. Merging cached changes at a later time, when affected pages are read into the buffer pool by other operations, avoids substantial random access I/O that would be required to read secondary index pages into the buffer pool from disk.
>
> In memory, the change buffer occupies part of the buffer pool. On disk, the change buffer is part of the system tablespace, where index changes are buffered when the database server is shut down.

#### Adaptive Hash Index

> The adaptive hash index feature enables InnoDB to perform more like an in-memory database on systems with appropriate combinations of workload and sufficient memory for the buffer pool without sacrificing transactional features or reliability. The adaptive hash index feature is enabled by the innodb_adaptive_hash_index variable, or turned off at server startup by --skip-innodb-adaptive-hash-index.
>
> Based on the observed pattern of searches, a hash index is built using a prefix of the index key. The prefix can be any length, and it may be that only some values in the B-tree appear in the hash index. Hash indexes are built on demand for the pages of the index that are accessed often.
>
> If a table fits almost entirely in main memory, a hash index can speed up queries by enabling direct lookup of any element, turning the index value into a sort of pointer. InnoDB has a mechanism that monitors index searches. If InnoDB notices that queries could benefit from building a hash index, it does so automatically.

#### Read-Ahead

> A read-ahead request is an I/O request to prefetch multiple pages in the buffer pool asynchronously, in anticipation that these pages will be needed soon. The requests bring in all the pages in one extent. InnoDB uses two read-ahead algorithms to improve I/O performance:
>
> - Linear read-ahead is a technique that predicts what pages might be needed soon based on pages in the buffer pool being accessed sequentially.
> - Random read-ahead is a technique that predicts when pages might be needed soon based on pages already in the buffer pool, regardless of the order in which those pages were read. If 13 consecutive pages from the same extent are found in the buffer pool, InnoDB asynchronously issues a request to prefetch the remaining pages of the extent.

### Miscellaneous

#### 数据安全

innodb_flush_log_at_trx_commit以及sync_binlog设为1，可以防止宕机时数据丢失。

> Even having transactional logging and strict flushing activated your database is still not protected from half-written pages. This may still happen during a server crash (for ex. power outage) when the page write operation was interrupted in the middle. So, you'll get a corrupted data, and it will be impossible to repair it from the redo log as there are only changes saved within redo.

#### Redo, undo and binary log

The redo log is a disk-based data structure used during crash recovery to correct data written by incomplete transactions.

An undo log is a collection of undo log records associated with a single read-write transaction. An undo log record contains information about how to undo the latest change by a transaction to a clustered index record.

The binary log contains “events” that describe database changes such as table creation operations or changes to table data. It also contains events for statements that potentially could have made changes (for example, a DELETE which matched no rows), unless row-based logging is used. The binary log also contains information about how long each statement took that updated data. The binary log has two important purposes:

- For replication, the binary log on a replication source server provides a record of the data changes to be sent to replicas. The source sends the events contained in its binary log to its replicas, which execute those events to make the same data changes that were made on the source.
- Certain data recovery operations require use of the binary log. After a backup has been restored, the events in the binary log that were recorded after the backup was made are re-executed. These events bring databases up to date from the point of the backup.

#### RR隔离下的幻读

在前一次快照读，第二次按条件update之后再读的情况下是会出现广义上的幻读的。

连续的快照读会因为MVCC不会出现幻读，连续的当前读会加Next-Key Lock，也不会出现幻读。

