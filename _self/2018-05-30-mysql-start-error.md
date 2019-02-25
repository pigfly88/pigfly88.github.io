---
layout: post
category: mysql
title:  记录一次MySQL启动报错debug过程
---

## 报错内容
mysql the control process exited with error code
Could not open unix socket lock file

service mysqld stop

## 查看错误日志

如果不知道错误日志的路径，可以查MySQL配置文件，如果MySQL配置文件的路径也不知道，使用下面的命令：

```shell
[root@vm11 ~]# mysqld --verbose --help | grep -A 1 'Default options'
Default options are read from the following files in the given order:
/etc/my.cnf /etc/mysql/my.cnf /usr/etc/my.cnf ~/.my.cnf
```

或者：

```shell
[root@vm11 ~]# whereis my.ini
my: /etc/my.cnf
```

定位到log-error这一样：

```shell
[root@vm11 ~]# cat /etc/my.cnf | grep 'log-error'

log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid
```

查看后面的错误：

```shell
tail -n 200 -f /var/log/mysqld.log
```

发现是权限问题，chown解决：

```shell
chown mysql:mysql mysql.sock
chown mysql:mysql mysql.sock.lock
```
