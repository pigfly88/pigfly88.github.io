---
layout: post
title:  MySQL事务的隔离级别
categories: mysql
---

隔离级别用来限定事务内的可见性，SQL标准定义了4种隔离级别，MySQL默认使用RR

1. READ UNCOMMITTED（读取未提交，以下简称RU）
    
	事务内可以读取到事务外未提交的内容

1. READ COMMITTED（读取已提交，以下简称RC）
    
	事务内可以读取到事务外已提交的内容

1. REPEATABLE READ（可重读，以下简称RR）
    
	事务内的同一条查询多次执行返回相同的数据

1. SERIALIZABLE（串行）

	对应一个记录会加锁，出现冲突的时候，后访问的事务必须等前一个事务执行完成才能继续执行

#### 事务隔离的实现

MySQL的默认隔离级别为RR，那么怎么去实现呢？怎么确保我每次查询的结果都是一样的呢，如果数据有多个版本，然后可以根据事务的版本找到相应版本的数据就好了。

#### MVCC
InnoDB 里面每个事务有一个唯一的事务 ID，叫作 transaction id。它是在事务开始的时候向 InnoDB 的事务系统申请的，是按申请顺序严格递增的。

当这个事务更新行记录的时候，会写入三个隐藏字段：

- row trx_id：当前事务的 transaction id 记为行的版本
- 回滚指针，指向 undolog
- 行ID

通过undolog，可以找到数据的历史版本，这就是MVCC（多版本并发控制）。

> MVCC指的是一种提高并发的技术。最早的数据库系统，只有读读之间可以并发，读写，写读，写写都要阻塞。引入多版本之后，只有写写之间相互阻塞，其他三种操作都可以并行，这样大幅度提高了InnoDB的并发度。在内部实现中，与Postgres在数据行上实现多版本不同，InnoDB是在undolog中实现的，通过undolog可以找回数据的历史版本。找回的数据历史版本可以提供给用户读(按照隔离级别的定义，有些读请求只能看到比较老的数据版本)，也可以在回滚的时候覆盖数据页上的数据。在InnoDB内部中，会记录一个全局的活跃读写事务数组，其主要用来判断事务的可见性。

#### 一致性视图

那么怎么通过当前事务 ID 去找到对应的数据版本呢？每个事务在启动瞬间会用一个数组记录下当前正在活跃的事务 ID，数组里面事务 ID 的最小值记为低水位，当前系统里面已经创建过的事务 ID 的最大值加 1 记为高水位。这个视图数组和高水位，就组成了当前事务的一致性视图。

![snapshot](/images/snapshot.png)

这样，对于当前事务的启动瞬间来说，一个数据版本的 row trx_id，有以下几种可能：
1. 如果落在绿色部分，表示这个版本是已提交的事务或者是当前事务自己生成的，这个数据是可见的；
2. 如果落在红色部分，表示这个版本是由将来启动的事务生成的，是肯定不可见的；
3. 如果落在黄色部分，那就包括两种情况：
    3.1 若 row trx_id 在数组中，表示这个版本是由还没提交的事务生成的，不可见；
    3.2 若 row trx_id 不在数组中，表示这个版本是已经提交了的事务生成的，可见。
    
一致性视图只会在 RR 与 RC 下才会生成，对于 RR 来说，一致性视图会在第一个查询语句的时候生成（或者在启动事务的时候指定：`start transaction with consistent snapshot` ）。而对于 RC 来说，每个查询语句都会重新生成视图。
    
#### 当前读和快照读

MySQL 使用 MVCC 机制，可以读取之前版本数据。这些旧版本记录不会且也无法再去修改，就像快照一样，所以我们将这种查询称为快照读。

- 快照读：简单SELECT，不加锁
- 当前读：读取最新的数据，不受隔离级别影响，加锁
    - SELECT ... LOCK IN SHARE MODE
    - SELECT ... FOR UPDATE
    - INSERT
    - DELETE
    - UPDATE
    
    - 快照读
    - SELECT * FROM TABLE WHERE ?

#### 为什么尽量不要使用长事务？

长事务意味着系统里面会存在很老的事务视图，在这个事务提交之前，回滚记录都要保留，这会导致大量占用存储空间。除此之外，长事务还占用锁资源，可能会拖垮库。

在开发过程中，少用长事务，如果无法避免，保证逻辑日志空间足够用，并且支持动态日志空间增长。可以在 information_schema 库的 innodb_trx 这个表中查询长事务，比如下面这个语句，用于查找持续时间超过 60s 的事务：
```sql
select * from information_schema.innodb_trx where TIME_TO_SEC(timediff(now(),trx_started))>60
```

#### 为什么MySQL默认的隔离级别是可重复读？
在MySQL5.0以前，如果采用读已提交，主从复制会存在bug。因为当时的binlog只支持statement格式，会导致master和slave的SQL执行顺序不一致，而最终导致主从数据不一致。

所以当时的解决办法就是把隔离级别设置为可重复读，在这个级别下，会通过间隙锁来保证执行的顺序。

后来binlog有了row格式，它不会出现这种执行顺序不一致的问题，InnoDB作者也提倡采用这种格式，这时候，采用读已提交的隔离级别，会有较好的并发性能。

### 查看和设置隔离级别

查看全局和当前会话的隔离级别：

```sql
mysql> SELECT @@global.tx_isolation, @@tx_isolation;
+-----------------------+----------------+
| @@global.tx_isolation | @@tx_isolation |
+-----------------------+----------------+
| REPEATABLE-READ       | READ-COMMITTED |
+-----------------------+----------------+
```

要为当前会话设置隔离级别，使用SET SESSION TRANSACTION语句：

```shell
mysql> SET SESSION TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
```

要修改全局配置，修改my.ini配置文件的```transaction-isolation```选项

## 各个隔离级别下的实例演示

### RU

mysql> SET SESSION TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;

![mysql read uncommitted](/images/mysql-read-uncommitted.png)

### RC

mysql> SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED;

![mysql read committed](/images/mysql-read-committed.png)

### RR

mysql> SET SESSION TRANSACTION ISOLATION LEVEL REPEATABLE READ;

![mysql repeatable read](/images/mysql-repeatable-read.png)

### SERIALIZABLE

mysql> SET SESSION TRANSACTION ISOLATION LEVEL SERIALIZABLE;

![mysql serializable](/images/mysql-serializable.png)

## 各个隔离级别下的锁策略

隔离级别不同，锁策略也不一样，锁定范围取决于查询条件使用到什么索引，下面就针对不同的查询条件来展开

### RC

- 主键索引。只锁定满足条件的主键索引记录
- 唯一索引。锁定满足条件的二级索引记录，并锁定对应的主键索引记录
- 普通索引。锁定满足条件的二级索引记录，并锁定对应的主键索引记录
- 没有索引。锁定主键索引的所有记录

### RR

- 主键索引。同RC
- 唯一索引。同RC
- 普通索引。锁定满足条件的二级索引记录，并锁定对应的主键索引记录，并在该二级索引记录范围内使用gap锁或next-key锁来防止其他事务插入索引范围内的数据，防止幻读
- 没有索引。锁定主键索引的所有记录，并在主键索引每条记录之间的间隙加上了gap锁

## 参考资料
- [MySQL实战45讲](https://time.geekbang.org/column/article/68319)
- [MySQL管理之道](https://item.jd.com/11973797.html)
- [MySQL 加锁处理分析](http://hedengcheng.com/?p=771)
- [MySQL refman innodb-transaction-isolation-levels](https://dev.mysql.com/doc/refman/8.0/en/innodb-transaction-isolation-levels.html)
- [InnoDB中的事务隔离级别和锁的关系](https://tech.meituan.com/innodb-lock.html)
- [淘宝数据库内核月报 - InnoDB 事务系统](http://mysql.taobao.org/monthly/2017/12/01/)
- [MySQL 可重复读，差点就让我背上了一个 P0 事故！](https://mp.weixin.qq.com/s?__biz=MzA3ODg3OTk4OA==&mid=2651097135&idx=4&sn=6ed14d6e1958d21062c96d8c8b5e4f8a&chksm=844c25b4b33baca258683c24f1e21ec0a0ac41befda73d74f61a3495b3415605df488f0d157c&mpshare=1&scene=1&srcid=&sharer_sharetime=1591102266856&sharer_shareid=9fbe450980474acdd3d83f0762a1e02a&key=fca09ad36b8e124bda8a01c184c807a9e96268465a880b47cf4e0479f66be5e8562ff3fa2a948644bfa1580eea96bea25ee127196190277f7276ce37e9a9b1ef39a1604bfd6aef93c00ed81a72f4cfdb&ascene=1&uin=NzM3MTI4MzQy&devicetype=Windows+10&version=62070158&lang=zh_CN&exportkey=A%2FlVQXadLfciJzHEG78IXEg%3D&pass_ticket=ke4%2Bc8h%2Fds7ijjvFttsu6VOdZYvjo1OufD74vIY1MXL9URGstOoMG0EUzQDErakC)