---
layout: post
category: "mysql"
title:  "MySQL重置密码"
---

```shell
# 关闭mysql
service mysqld stop
mysqld --skip-grant-tables --user=root

# 新开一个窗口
mysql
mysql> FLUSH PRIVILEGES;
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY '123';
```

> [mysql-resetting-permissions](https://dev.mysql.com/doc/refman/5.7/en/resetting-permissions.html)