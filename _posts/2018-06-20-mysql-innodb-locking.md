---
layout: post
title:  InnoDB中的锁
categories: mysql
---

## 锁的类型

- 共享锁和排它锁
- 意向锁
- 记录锁
- 间隙锁
- Next-Key锁
- 插入意向锁
- 自增锁

### 共享锁和排它锁

共享锁：拿到共享锁(S)的事务可以读取这一行

排它锁：拿到排它锁(X)的事务可以更新或删除这一行

假设事务T1拿到了第r行的S锁，那么另外一个事务T2可以获取S锁，但不能获取X锁

假设事务T1拿到了第r行的X锁，那么另外一个事务T2不能获取S锁和X锁

```sql
mysql> SHOW VARIABLES LIKE "%innodb_status%";
+----------------------------+-------+
| Variable_name              | Value |
+----------------------------+-------+
| innodb_status_output       | OFF   |
| innodb_status_output_locks | OFF   |
+----------------------------+-------+

mysql> SET GLOBAL innodb_status_output=ON;
mysql> SET GLOBAL innodb_status_output_locks=ON;
```

<table>
<thead>
<tr>
<th>T1</th>
<th>T2</th>
</tr>
</thead>
<tbody>
<tr>
<td>start transaction;</td>
<td>start transaction;</td>
</tr>
<tr>
<td>
select * from user where id=1 lock in share mode;<br>
+----+--------+-------------+<br>
| id | name   | phone       |<br>
+----+--------+-------------+<br>
|  1 | pigfly | 13714148963 |<br>
+----+--------+-------------+
</td>
<td></td>
</tr>
<tr>
<td></td>
<td>
select * from user where id=1;<br>
+----+--------+-------------+<br>
| id | name   | phone       |<br>
+----+--------+-------------+<br>
|  1 | pigfly | 13714148963 |<br>
+----+--------+-------------+
</p>
</td>
</tr>
<tr>
<td></td>
<td></td>
</tr>
</tbody>
</table>


### 意向锁

- 事务在获取S锁之前，必须先获取表的IS锁
- 事务在获取X锁之前，必须先获取表的IX锁

所以，如果存在IS锁，说明某个事务接下来会加S锁，如果存在IX锁，说明事务接下来会加X锁

事务T1申请S锁
事务T1获取IS锁

事务T2申请