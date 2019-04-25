---
layout: post
title:  MyISAM加锁分析
categories: mysql
---

### 为什么加锁
你正在读着你喜欢的女孩递给你的信，看到一半的时候，她的好闺蜜过来瞄了一眼（假设她会隐身术，你看不到她），她想把“我很喜欢你”改成“我不喜欢你”，刚把“很”字擦掉，“不”字还没写完，只写了一横一撇，这时候你正读到这个字，她怕你察觉到也就没继续往下写了，这时候你读到的这句话就是“我丆喜欢你”，这是什么鬼？！这位闺蜜乐了：没错，确实是鬼在整蛊你呢，嘿嘿！

数据库也会闹鬼吗？很有可能！假设会话1正在读取表里的一条记录（还没读取完），另一个会话2突然插队过来更新表里的同一条记录（还没更新完），那么会话1拿到的数据就可能是错误的（还没更新完的内容和原内容混在一起，造成乱码，就像上面的“我丆喜欢你”）。

怎么避免这种情况呢？加锁，当有一个人在读的时候，别人能读不能写，当有一个人在写的时候，别人不能读和写。

所以，加锁是为了在并发操作的时候，能够确保数据的完整性和一致性。

### 加锁的规则

MyISAM锁的粒度是**表级锁**，在执行查询（SELECT）之前，尝试在表上面加读锁，在执行更新（UPDATE,DELETE,INSERT）之前，尝试在表上面加写锁。

加写锁：

如果在表上没有锁（读锁和写锁），在它上面放一个写锁。
否则，把锁定请求放在写锁定队列中。

加读锁：

如果在表上没有写锁定，把一个读锁定放在它上面。
否则，把锁定请求放在读锁定队列中。

优先级：

**当一个锁定被释放时，锁定优先被写锁定队列中的线程得到，然后是读锁定队列中的线程。**这意味着如果有大量的写操作，读操作将会一直等待，直到写完成。可以通过以下命令看到加锁的情况：

```sql
SHOW STATUS LIKE 'table%';
+-----------------------+-------+
| Variable_name         | Value |
+-----------------------+-------+
| Table_locks_immediate | 42    |
| Table_locks_waited    | 3     |
+-----------------------+-------+
```

Table_locks_immediate是加锁立刻执行成功的次数，Table_locks_waited是造成等待的加锁次数。另外，可以通过LOW_PRIORITY来[改变优先级](https://mariadb.com/kb/zh-cn/high_prioritylow_priority/)。

### 实例分析

开一个会话窗口1，输入下面的语句执行：
```sql
CREATE TABLE `users`(
`id` int(10) unsigned NOT NULL AUTO_INCREMENT,
`name` varchar(15) NOT NULL,
PRIMARY KEY (`id`)
)ENGINE=MYISAM DEFAULT CHARSET=utf8 COMMENT='用户';

INSERT INTO `users` VALUES (null, 'pigfly'),(null,'zhupp');
```

为了模拟，我们手动执行[LOCK TABLES](http://tool.oschina.net/uploads/apidocs/mysql-5.1-zh/sql-syntax.html#lock-tables)语句把表锁住：
```sql
LOCK TABLES `users` READ LOCAL;
SELECT * FROM `users`;
UPDATE `users` SET name='aa' where id=1;
```
SELECT正常返回，UPDATE报错了，原因是当前表加了读锁，则当前会话只能执行读操作，不能执行更新操作。

新开一个会话窗口2：
```sql
INSERT INTO `users` VALUES (null, 'zhupp');
UPDATE `users` SET name='xxx' where id=1;
```
可以看到插入执行成功，但是UPDATE操作被窗口1加的读锁阻塞了，我们回到窗口1执行：
```sql
UNLOCK TABLES;
```
这时候窗口2的更新语句马上返回更新成功了。

为什么插入不会被读锁阻塞呢？原因是当表加了读锁并且表不存在空闲块的时候（删除或者更新表中间的记录会导致空闲块，OPTIMIZE TABLE可以清除空闲块），MYISAM默认允许其他线程从表尾插入。可以通过改变系统变量concurrent_insert（并发插入）的值来控制并发插入的行为。
```sql
SHOW VARIABLES LIKE 'concurrent%';
+-------------------+-------+
| Variable_name     | Value |
+-------------------+-------+
| concurrent_insert | AUTO  |
+-------------------+-------+
```

Value的值：

- NEVER(0): 不允许并发插入
- AUTO(1): 表里没有空行时允许从表尾插入（默认）
- ALWAYS(2): 任何时候都允许并发插入

注意：锁表的时候加了**LOCAL关键字**表示允许走并发插入的逻辑，具体是否可以并发插入还需要看是否满足concurrent_insert指定的条件，只有手动锁表的时候才需要指定LOCAL关键字。

测试一下当表里有空闲块的情况，窗口1执行：
```sql
DELETE FROM `users` WHERE id=1;
LOCK TABLES `users` READ LOCAL;
```

然后在窗口2执行：
```sql
INSERT INTO `users` VALUES (null, 't1');
```

果然被阻塞了。我们把并发插入的值改成2试试，在窗口1执行：
```sql
UNLOCK TABLES;
SET GLOBAL concurrent_insert=2;
DELETE FROM `users` WHERE id=2;
LOCK TABLES `users` READ LOCAL;
```

然后在窗口2执行：
```sql
INSERT INTO `users` VALUES (null, 't2');
SELECT * FROM `users`;
```
这一次没有被阻塞，插入成功了。

### 表级锁的特点

开销小、加锁快、不会产生死锁，锁定力度大，发生锁冲突的概率最高，不适合高并发场景。

### 性能优化

1. 对于并发插入，一般默认配置AUTO就可以了，如果有大量插入操作，可以把concurrent_insert设置为2，然后定期在流量低峰期执行OPTIMIZE TABLE来清除空闲块。
2. 调整优先级。
3. 在大量更新操作前手动锁表，这样锁表只执行了一次，不然每执行一次更新就锁一次表。
4. 存在大量更新操作造成等待，又要兼顾查询的时候，给max_write_lock_count设置一个低值，在写锁达到一定数量时允许执行挂起的读请求。

### 参考资料
- [MySQL锁定事宜](http://tool.oschina.net/uploads/apidocs/mysql-5.1-zh/optimization.html#locking-issues)
- [高性能MySQL](https://book.douban.com/subject/23008813/)
- [PHP核心技术与最佳实践](https://book.douban.com/subject/20370984/)