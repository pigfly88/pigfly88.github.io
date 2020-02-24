---
layout: post
title:  MySQL索引初探
categories: mysql
---

## 什么是索引？

索引的出现其实就是为了提高数据查询的效率，就像书的目录一样。

索引本身也占据磁盘空间，更新和删除数据的时候也要对索引进行维护。

## 索引的类型

- 哈希：KV结构，只适用于等值查询，仅MEMORY引擎支持。
- B+树：大多数索引都使用B树存储，比如PRIMARY KEY, UNIQUE, INDEX, FULLTEXT。

![](/images/index-hash-vs-btree.jpg)

### B树和B+树

![](/images/btree-vs-b+tree.png)
- B树
	- 叶子节点，非叶子节点，都存储数据；
	- 中序遍历，可以获得所有节点。
- B+树
	- 数据只存储在叶子节点上；
	- 叶子之间，增加了链表，获取所有节点，不再需要中序遍历。

B+树的优势：

- 范围查找，定位min与max之后，中间叶子节点，就是结果集，不用中序回溯；
- 非叶子节点，不存储实际记录，在相同内存的情况下，B+树能够存储更多索引。

## 索引的法则

1. 如果有多个索引，MySQL会优先使用行数最少的索引

1. 最左前缀。例如有INDEX(col1, col2, col3)，那么在(col1)，(col1, col2)，(col1, col2, col3)上执行WHERE，GROUP BY，ORDER BY操作的时候可以使用索引

1. 在进行列比较或者表连接的时候，如果列有相同的数据类型和大小以及字符编码，能更好地使用索引

1. MIN(),MAX()索引列直接返回
    例如有索引INDEX(col1, col2)，考虑以下查询：
    ```sql
    SELECT MIN(col2), MAX(col2) FROM tbl WHERE col1 = 12;
    ```
    MySQL通过(col1)索引找到节点，然后第一个(col1, col2)里面的col2就是MIN(col2)，最后一个(col1, col2)里面的col2就是MAX(col2)，因为本来就是排好序的，所以不需要在经过计算就能直接返回

1. 索引覆盖
    例如有索引INDEX(col1, col2, col3)，考虑以下查询：
    ```sql
    SELECT col1, col2 FROM tbl WHERE col1 = 12;
    ```
    因为索引INDEX(col1, col2, col3)包含了需要查询的列(col1, col2)，所以MySQL在通过索引找到匹配的时候，不需要再去找对应的行来找到其他列的值


## 一些实例

下载MySQL官网提供的测试数据库

[Employees Sample Database](https://dev.mysql.com/doc/employee/en/)


ALTER TABLE employees ADD INDEX ln_fn_bd (last_name, first_name, birth_date);

mysql> EXPLAIN SELECT * FROM employees WHERE last_name = 'Piveteau';
+----+-------------+-----------+------------+------+---------------+----------+---------+-------+------+----------+-------+
| id | select_type | table     | partitions | type | possible_keys | key      | key_len | ref   | rows | filtered | Extra |
+----+-------------+-----------+------------+------+---------------+----------+---------+-------+------+----------+-------+
|  1 | SIMPLE      | employees | NULL       | ref  | ln_fn_bd      | ln_fn_bd | 18      | const |  198 |   100.00 | NULL  |
+----+-------------+-----------+------------+------+---------------+----------+---------+-------+------+----------+-------+


## 参考资料
1. [从B 树、B+ 树、B* 树谈到R 树](https://blog.csdn.net/v_JULY_v/article/details/6530142)
1. [算法可视化](https://www.cs.usfca.edu/~galles/visualization/BTree.html)
1. [MySQL实战45讲](https://time.geekbang.org/column/intro/139)
1. [数据库索引，到底是什么做的？](https://mp.weixin.qq.com/s?__biz=MjM5ODYxMDA5OQ==&mid=2651961486&idx=1&sn=b319a87f87797d5d662ab4715666657f&chksm=bd2d0d528a5a84446fb88da7590e6d4e5ad06cfebb5cb57a83cf75056007ba29515c85b9a24c&scene=21#wechat_redirect)