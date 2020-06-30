---
layout: post
title:  SQL中过滤条件放在on和where中的区别
categories: mysql
---


今天接到蚂蚁金服的电面，问了sql中过滤条件放在on和where中的区别，当时满脑子是inner join，觉得没区别啊。后来才想起来，连接查询除了inner join还有right join，left join。汗呐，当时还是太紧张了。这里做一下记录吧。

join过程可以这样理解：首先两个表做一个笛卡尔积，on后面的条件是对这个笛卡尔积做一个过滤形成一张临时表，如果没有where就直接返回结果，如果有where就对上一步的临时表再进行过滤。下面看实验：

先准备两张表：


### 先执行inner join

```sql
select * from person p inner join account a on p.id=a.id and p.id!=4 and a.id!=4;
```

![](/images/mysql-on-vs-where-2.png)

```sql
select * from person p inner join account a on p.id=a.id where p.id!=4 and a.id!=4;
```

![](/images/mysql-on-vs-where-2.png)

结果没有区别，前者是先求笛卡尔积然后按照on后面的条件进行过滤，后者是先用on后面的条件过滤，再用where的条件过滤。


### 再看看左连接left join

```sql
select * from person p left join account a on p.id=a.id and p.id!=4 and a.id!=4;
```

![](/images/mysql-on-vs-where-3.png)

这下看出来不对了，id为4的记录还在，这是由left join的特性决定的，**使用left join时on后面的条件只对右表有效**（可以看到右表的id=4的记录没了）

```sql
select * from person p left join account a on p.id=a.id where p.id!=4 and a.id!=4;
```

![](/images/mysql-on-vs-where-4.png)

where的过滤作用就出来了。。。

右连接的原理是一样的。。


到这里就真相大白了：**inner join中on和where没区别，右连接和左连接就不一样了**。


> 版权声明：本文为CSDN博主「古月慕南」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：[SQL中过滤条件放在on和where中的区别](https://blog.csdn.net/u013468917/java/article/details/61933994)