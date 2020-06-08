---
layout: post
title:  MySQL索引初探
categories: mysql
---

## 什么是索引？

索引的出现其实就是为了提高数据查询的效率，就像书的目录一样。

索引本身也占据磁盘空间，更新和删除数据的时候也要对索引进行维护。

## 索引的类型

### 哈希
KV结构，只适用于等值查询，仅MEMORY引擎支持。

### 二叉查找树

![](/images/bst.webp)

从图中可以看到，我们为user表（用户信息表）建立了一个二叉查找树的索引。图中的圆为二叉查找树的节点，节点中存储了键(key)和数据(data)。

键对应user表中的id，数据对应user表中的行数据。二叉查找树的特点就是左子节点的键值小于父节点的键值，右子节点的键值大于父节点的键值。 顶端的节点我们称为根节点，没有子节点的节点我们称之为叶节点。 

### 平衡二叉查找树
二叉查找树有可能很不平衡，最坏的情况是退化成一个链表，导致查找效率低下：

![](/images/bstnb.webp)

为了解决这个问题，我们需要保证二叉查找树一直保持平衡，就需要用到平衡二叉树了。 


平衡二叉树又称AVL树，在满足二叉查找树特性的基础上，要求每个节点的左右子树的高度差不能超过1。 

![](/images/avl-tree.webp)

### B树

索引不光维护在内存中，还要持久化到磁盘里，在数据量比较大的情况下，平衡二叉树的高度会很高，而我们每查找一次数据就需要从磁盘中读取一个节点，也就是我们说的一个磁盘块，我们都知道平衡二叉树可是每个节点只存储一个键值和数据的，这种情况会进行很多次磁盘 IO，我们查找数据的效率将会极低！所以我们要想办法在一个磁盘块里尽量多放一些节点数据进去，那么二叉树就要改造成N叉树也就是树了。

![](/images/b-tree.webp)

从上图可以看出，B树相对于平衡二叉树，每个节点存储了更多的键值(key)和数据(data)，并且每个节点拥有更多的子节点，子节点的个数一般称为阶，上述图中的B树为3阶B树，高度也会很低。

以 InnoDB 的一个整数字段索引为例，这个 N 差不多是 1200。这棵树高是 4 的时候，就可以存 1200 的 3 次方个值，这已经 17 亿了。考虑到树根的数据块总是在内存中的，一个 10 亿行的表上一个整数字段的索引，查找一个值最多只需要访问 3 次磁盘。其实，树的第二层也有很大概率在内存中，那么访问磁盘的平均次数就更少了。 

### B+树

B+树是对B树的进一步优化。让我们先来看下B+树的结构图：

![](/images/b+tree.webp)

1. B+树非叶子节点上是不存储数据的，仅存储键值，而B树节点中不仅存储键值，也会存储数据。之所以这么做是因为在数据库中页的大小是固定的，innodb中页的默认大小是16KB。如果不存储数据，那么就会存储更多的键值，相应的树的阶数（节点的子节点树）就会更大，树就会更矮更胖，如此一来我们查找数据进行磁盘的IO次数有会再次减少，数据查询的效率也会更快。另外，B+树的阶数是等于键值的数量的，如果我们的B+树一个节点可以存储1000个键值，那么3层B+树可以存储1000×1000×1000=10亿个数据。一般根节点是常驻内存的，所以一般我们查找10亿数据，只需要2次磁盘IO。 

2. 因为B+树索引的所有数据均存储在叶子节点，而且数据是按照顺序排列的。那么B+树使得范围查找，排序查找，分组查找以及去重查找变得异常简单。而B树因为数据分散在各个节点，要实现这一点是很不容易的。

总结一下B树和B+树的区别：

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

1. 最左前缀。例如有索引`INDEX(col1, col2, col3)`，那么在(col1)，(col1, col2)，(col1, col2, col3)上执行WHERE，GROUP BY，ORDER BY操作的时候可以使用索引。`where col1 like 'a%'`这种方式也是满足最左前缀原则（最左字段上的左边字符串）的

1. 在进行列比较或者表连接的时候，如果列有相同的数据类型和大小以及字符编码，能更好地使用索引

1. MIN(),MAX()索引列直接返回
    例如有索引`INDEX(col1, col2)`，考虑以下查询：

    ```sql
    SELECT MIN(col2), MAX(col2) FROM tbl WHERE col1 = 12;
    ```

    MySQL通过(col1)索引找到节点，然后第一个(col1, col2)里面的col2就是MIN(col2)，最后一个(col1, col2)里面的col2就是MAX(col2)，因为本来就是排好序的，所以不需要在经过计算就能直接返回

1. 索引覆盖
    例如有索引`INDEX(col1, col2, col3)`，考虑以下查询：

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
2. [不准犹豫！再有人问你为什么MySQL用B+树做索引，就把这篇文章发给她](https://mp.weixin.qq.com/s?__biz=Mzg2NzA4MTkxNQ==&mid=2247486251&idx=1&sn=296f07b65b5a73a15337541fb4bc6572&chksm=ce4040fff937c9e92f1046d0fc0ce7614fa08cb46c67e38c185005260537e35e822c354610a5&mpshare=1&scene=1&srcid=1109YA6CrUq3lrDgzlGVewX0&sharer_sharetime=1591329228315&sharer_shareid=9fbe450980474acdd3d83f0762a1e02a&key=f2043c48cbd2a4d3c0ff2a30219ecd5ca01bb1c8bfe5977f5928085d3233e6780193f79897136bda99557aee1629ab85e28fe5f06ce3a43c6ef2d35d80acbfaee3282ab90553dd553c429566338a0cf7&ascene=1&uin=NzM3MTI4MzQy&devicetype=Windows+10&version=62070158&lang=zh_CN&exportkey=AxqjzaOs48f8jhXd4dL2QeA%3D&pass_ticket=azMM9aEFmcx%2FMYOKlXEr64xduf2HdGX6hJGdpa5NtLXFpfd0FrEgVOoUhyeHtcmq)