---
layout: post
category: mysql
title:  MySQL复制初探
---

当需要分摊数据库查询压力的时候，要部署多台MySQL来实现读写分离，比如一主多从，主可以读和写，从只能读，这个时候就需要用到复制来让这几台数据库的数据同步。

## 复制原理
MySQL支持两种复制方案：
- 基于语句复制
- 基于行复制

两种方式都是通过在主服务器上面写二进制日志，从服务器拉取日志然后重放。
![mysql replication](/images/mysql-replication.jpg)

下面我们就来简单体验一下MySQL的复制，首先在主服务器上面创建一个用于测试的数据库：
```shell
mysql> CREATE DATABASE IF NOT EXISTS `shop`;
mysql> USER `shop`;
mysql> CREATE TABLE `orders` (
 `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
 `orderno` char(60) NOT NULL DEFAULT '',
 `uid` int(11) unsigned NOT NULL DEFAULT '0',
 `amount` decimal(8,2) unsigned DEFAULT '0.00',
 `status` tinyint(2) unsigned NOT NULL DEFAULT '0' ,
 PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

mysql> INSERT INTO `orders` VALUES (null, '2018053001', 1, 99.8, 1);
```

在主服务器上创建一个复制账号，其中repl是用户名，@后面可以配置IP限制：
```shell
mysql> GRANT REPLICATION SLAVE,REPLICATION CLIENT ON *.* TO repl@'192.168.56.%' IDENTIFIED BY '%678NJImko';
```

在主服务器上面打开二进制日志并且配置server id：
```shell
[root@vm11 ~]# mysqld --verbose --help | grep -A 1 'Default options'
Default options are read from the following files in the given order:
/etc/my.cnf /etc/mysql/my.cnf /usr/etc/my.cnf ~/.my.cnf
[root@vm11 ~]# vi /etc/my.cnf
log_bin=mysql-bin
server_id=11

#配置好后重启musql
service mysqld restart
```

从服务器配置：
```shell
log_bin=mysql-bin
server_id=10
relay_log=mysql-relay-bin
log_slave_updates=1
read_only=1

#配置好后重启musql
service mysqld restart
```

# 关闭防火墙
systemctl stop firewalld
systemctl disable firewalld
或者：
/etc/rc.d/init.d/iptables stop

mysql> show master status;
+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000001 |      154 |              |                  |                   |
+------------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)

### 开始复制
在从服务器上启动复制：
```shell
mysql> CHANGE MASTER TO MASTER_HOST='192.168.56.11',
    -> MASTER_USER='repl',
    -> MASTER_PASSWORD='%678NJImko',
    -> MASTER_LOG_FILE='mysql-bin.000002',
    -> MASTER_LOG_POS=0;

mysql> START SLAVE;
mysql> SHOW SLAVE STATUS;

Fatal error: The slave I/O thread stops because master and slave have equal MySQL server UUIDs; these UUIDs must be different for replication to work.

mysql> SHOW VARIABLES LIKE '%uuid%';
+---------------+--------------------------------------+
| Variable_name | Value                                |
+---------------+--------------------------------------+
| server_uuid   | 1d9ae556-64b9-11e8-bc08-0800270d06eb |
+---------------+--------------------------------------+

https://dev.mysql.com/doc/refman/5.7/en/replication-options.html

查文档看到启动MySQL的时候会自动生成这个文件，所以删掉，然后重启MySQL，问题解决

mysql> SHOW VARIABLES LIKE '%datadir%';
+---------------+-----------------+
| Variable_name | Value           |
+---------------+-----------------+
| datadir       | /var/lib/mysql/ |
+---------------+-----------------+

[root@vm10 mysql]# cd /var/lib/mysql
[root@vm10 mysql]# mv auto.cnf auto.cnf.bak
[root@vm10 mysql]# service mysqld restart


## ab压测
ab -c 100 -n 5000 http://192.168.56.11/test.php