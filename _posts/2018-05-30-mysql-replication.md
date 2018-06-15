## 复制原理
![mysql replication](/images/mysql-replication.jpg)

mysql> CREATE DATABASE IF NOT EXISTS `shop`
    -> DEFAULT CHARSET = utf8
    -> DEFAULT COLLATE = utf8_general_ci;


mysql> CREATE TABLE `orders` (
 `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
 `orderno` char(60) NOT NULL DEFAULT '',
 `uid` int(11) unsigned NOT NULL DEFAULT '0',
 `amount` decimal(8,2) unsigned DEFAULT '0.00',
 `status` tinyint(2) unsigned NOT NULL DEFAULT '0' ,
 PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

insert into `orders` values (null, '2018053001', 1, 99.8, 1);

mysql> GRANT REPLICATION SLAVE,REPLICATION CLIENT ON *.* TO repl@'192.168.56.%' IDENTIFIED BY '%678NJImko';

[root@vm11 ~]# mysqld --verbose --help | grep -A 1 'Default options'
Default options are read from the following files in the given order:
/etc/my.cnf /etc/mysql/my.cnf /usr/etc/my.cnf ~/.my.cnf
[root@vm11 ~]# vi /etc/my.cnf
log_bin=mysql-bin
server_id=11

service mysqld restart
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

### 从服务器配置

log_bin=mysql-bin
server_id=10
relay_log=mysql-relay-bin
log_slave_updates=1
read_only=1

service mysqld restart

### 开始复制
mysql> CHANGE MASTER TO MASTER_HOST='192.168.56.11',
    -> MASTER_USER='repl',
    -> MASTER_PASSWORD='%678NJImko',
    -> MASTER_LOG_FILE='mysql-bin.000002',
    -> MASTER_LOG_POS=0;

insert into `orders` values (null, '2018053002', 2, 39, 1);

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