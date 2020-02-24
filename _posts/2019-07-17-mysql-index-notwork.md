---
layout: post
title:  索引失效的原因
categories: mysql
---

```sql
CREATE TABLE `tbl` (
	`id` int(10) NOT NULL PRIMARY KEY AUTO_INCREMENT,
	`sid` varchar(20) NOT NULL DEFAULT '',
	KEY `sid`(`sid`)
)ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

CREATE TABLE `tbl_2` (
  `id` int(10) NOT NULL AUTO_INCREMENT,
  `name` varchar(20) NOT NULL DEFAULT '',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=3 DEFAULT CHARSET=utf8mb4;

INSERT INTO `tbl` VALUES (NULL, '1'),(NULL, '3'),(NULL, '5'),(NULL, '666');
INSERT INTO `tbl_2` VALUES (NULL, 'a'),(NULL, 'b');
```

- 对字段做函数操作：函数操作破坏了索引的有序性

```sql
EXPLAIN SELECT * FROM `tbl` WHERE MONTH(updated_at)=7;
```

- 字段类型转换

```sql
EXPLAIN SELECT * FROM `tbl` WHERE `sid`=1;
```

<table>
    <tr>
        <td>type</td>
		<td>rows</td>
    </tr>
	<tr>
        <td>ALL</td>
		<td>4</td>
    </tr>
</table>

在MySQL中，字符串和数字做比较的话，是会把字符串转换成数字的。

因为 sid 是 varchar 类型，所以当查询条件等于整数的时候，相当于执行了下面的语句：

```sql
select * from tbl where  CAST(sid AS signed int) = 1
```

也就是说触发了上面说到的函数操作，使索引失效。

连表查询也可能出现这种情况：

```sql
EXPLAIN SELECT * FROM tbl_2 t2 JOIN tbl t1
ON t2.id = t1.sid
WHERE t2.id = 1;
```

由于t2表的id是int类型，t1表的sid是varchar类型，所以会触发函数操作，相当于执行了下面的语句：

```sql
EXPLAIN SELECT * FROM tbl_2 t2 JOIN tbl t1
ON t2.id = CAST(t1.sid AS signed int)
WHERE t2.id = 1;
```

- 编码类型转换


```
