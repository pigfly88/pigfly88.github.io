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

每条记录在更新的时候都会同时记录一条回滚操作。同一条记录在系统中可以存在多个版本，这就是数据库的多版本并发控制（MVCC）。

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

## 快照读和当前读

- 快照读，不加锁
    - SELECT * FROM TABLE WHERE ?

- 当前读，加锁
    - SELECT * FROM TABLE WHERE ? LOCK IN SHARE MODE;
    - SELECT * FROM TABLE WHERE ? FOR UPDATE;
    - INSERT ?
    - DELETE ?
    - UPDATE ?

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