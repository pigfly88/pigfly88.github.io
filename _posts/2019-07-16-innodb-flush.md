---
layout: post
title:  InnoDB刷盘策略
categories: mysql
---

InnoDB的特点是会把数据尽量捞到内存，然后直接操作内存（buffer pool），数据更新是顺序写入redo log的，所以比直接读取磁盘快。

所以，平时执行很快的更新操作，其实就是在写内存和日志，而 MySQL 偶尔“抖”一下的那个瞬间，可能就是在刷脏页（flush）。

什么情况出发flush：

- 内存写满了。当要操作的数据页没有在内存的时候，就必须到缓冲池中申请一个数据页，这时会优先把内存的脏页刷到磁盘来腾出空间，这时候只能把最久不使用的数据页从内存中淘汰掉：如果要淘汰的是一个干净页，就直接释放出来复用；但如果是脏页呢，就必须将脏页先刷到磁盘，变成干净页后才能复用。
- redo log写满了。这时需要先把redo log刷到磁盘。

调整刷盘策略：

- innodb_io_capacity：刷盘速度，建议设置成磁盘的IOPS（可通过fio工具分析）。
- innodb_max_dirty_page_pct：脏页比例上限。

参考资料：
- [为什么我的MySQL会“抖”一下？](https://time.geekbang.org/column/article/71806)
- [Optimizing InnoDB Disk I/O](https://dev.mysql.com/doc/refman/5.7/en/optimizing-innodb-diskio.html)