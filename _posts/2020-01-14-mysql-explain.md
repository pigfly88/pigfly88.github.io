---
layout: post
category: "mix"
title:  "MySQL explain解析"
---

### 数据准备
```sql
create table user (
	id int primary key,
	name varchar(20)，
	age unsigned tinyint,
	index(age)
)engine=innodb;

create table user_ex (
	id int primary key,
	age unsigned tinyint,
	index(age)
)engine=innodb;

insert into user values(1,'shenjian',18),(2,'zhangsan',19),(3,'lisi',20);
insert into user_ex values(1,18),(2,19),(3,20),(4,19);
```

### type

- system：系统表，少量数据，往往不需要进行磁盘IO；
	
	explain select * from mysql.time_zone;
	
	select * from (select * from user where id=1) tmp;
	
- const：主键或者唯一索引常量连接；

	select * from user where id=1;
	
- eq_ref：主键索引(primary key)或者非空唯一索引(unique not null)等值扫描。对于前表的每一行，后表只有一行被扫描；

	select * from user,user_ex where user.id=user_ex.id;
	
- ref：为非唯一普通索引等值扫描。对于前表的每一行，后表可能有多于一行的数据被扫描；

	select * from user,user_ex where user.age=user_ex.age;
	
- range：范围扫描；

	select * from user where id between 1 and 4;

	select * from user where idin(1,2,3);

	select * from user where id>3;

- index：走索引的全表扫描，需要扫描索引上的全部数据；

	select count (*) from user;
	
- ALL：全表扫描(full table scan)，没有用到索引；

	select * from user where name='zzh';
	
	possible_keys：NULL

 
possible_keys：可能使用的索引

key：实际使用的索引

ref：哪些列，或者常量用于查找索引上的值。

rows：扫描的行数