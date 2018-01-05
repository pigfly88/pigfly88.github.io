---
layout: post
title:  安装Redis
categories: nosql
keywords: install Redis
---

## 介绍
- redis相比memcache能够支持更多的数据类型：string（字符串）、list（链表）、set（集合）、zset（有序集合）、hash（散列表）
- 提供原子性的操作，保证数据的一致性
- 数据快照，通过定期写入rdb文件来保证数据不丢失
- 虚拟内存

### 字符串结构体
redis把所有数据都通过SDS（simple dynamic string简单动态字符串）保存成字符串类型
```c
struct sdshdr {
    // 存储字符串的字符数组
    char buff[];

    // buff数组长度
    unsigned int len;

    // 可用字节
    unsigned int free;
};
```

### 虚拟内存
跟MongoDB不同，Redis没有使用内存映射，而是自己实现了[虚拟内存](http://www.redis.cn/topics/internals-vm)

## 安装
```shell
$ cd /usr/local/src
$ wget http://download.redis.io/releases/redis-4.0.2.tar.gz
$ tar -zxvf redis-4.0.2.tar.gz
$ mv redis-4.0.2 /usr/local/redis
$ cd /usr/local/redis/
$ make
$ src/redis-server
 基本命令
$ /usr/local/redis/src/redis-cli

---------字符串---------
set name pigfly
OK
get name
pigfly

set age 23 EX 10 #10秒后过期
OK
get age
23
get age
$-1 #10秒后已过期，返回-1

set name pig NX #不存在才设置
$-1 #因为存在所以返回失败
set name pig XX #存在才设置
OK
get name
pig

---------链表---------
lpush msg "hello!"
:1
lpush msg "where are you?"
:2

lrange msg 0 -1
where are you?
hello!

rpush msg "i'm here!"
:3
lrange msg 0 -1
where are you?
hello!
i'm here!

lindex msg 1
hello!

lpop msg
where are you?

blpop news 0 #阻塞式
lrange msg 0 -1
lpush msg "hey"
news
bye~
hello!
i'm here!
:3

---------散列表---------
hset userinfo name "pigfly"
:1
hget userinfo name
pigfly
hgetall userinfo
name
pigfly

hmset userinfo age 28 sex 1
OK
hgetall userinfo
name
pigfly
age
28
sex
1

hincrby userinfo age 1
:29
hget userinfo age
29
hmget userinfo name sex
pigfly
1
hsetnx userinfo age 32 #字段不存在时才设置
:0
hget userinfo age
28
hsetnx userinfo qq "1315829"
:1
hget userinfo qq
1315829

---------集合---------
sadd friends:299 1 2 3 7 8 9
:6
smembers friends:299
1
2
3
7
8
9

scard friends:299
:6
sadd friends:300 1 7 10 11
:4
sdiff friends:299 friends:300
2
3
8
9
sdiff friends:300 friends:299
10
11

sismember friends:300 1
:1
sismember friends:300 6
:0

---------有序集合---------
zadd hotlist 1 "xx"
:1
zadd hotlist 5 "yy" 2 "bb"
:2
zcard hotlist
:3
zrange hotlist 0 -1
xx
bb
yy

zrevrange hotlist 0 -1
yy
bb
xx
zincrby hotlist 5 "bb"
7
zrevrange hotlist 0 -1
bb
yy
xx
```

## 安装php客户端
```shell_session
$ cd /usr/local/src
$ wget http://pecl.php.net/get/redis-3.1.4.tgz
$ tar -zxvf redis-3.1.4.tgz
$ cd redis-3.1.4
$ phpize
$ ./configure
$ make && make install
#修改php配置文件
$ vi /usr/local/php/lib/php.ini
extension=redis.so
$ apachectl restart
```