---
layout: post
title:  安装MongoDB
date:   2017-12-18
categories: nosql
---

## 介绍
MongoDB是一个基于分布式、面向文档存储的非关系型数据库。它是一个介于关系数据库和非关系数据库之间的产品，是非关系数据库当中功能最丰富，最像关系数据库的。由 C++ 语言编写，旨在为 WEB 应用处理大量数据并提供可扩展的高性能数据存储解决方案

### 特点
- 无表结构
- 可以像关系型数据库那样完成复杂的查询操作
- 也能添加索引
- 不支持JOIN查询和事务
- 创建和更新数据的时候不会实时写入硬盘

### 如何保存数据
把数据和数据结构以BSON（binary encoded serialization of JSON-like structure 二进制编码序列化类JSON结构）形式保存，并把它作为值和特定的键进行关联

### 数据库用语

关系型数据库 | 面向文档数据库
----|------
数据库  | 数据库
表 | 集合
记录 | 文档

## 安装MongoDB
```shell_session
$ cd /etc/yum.repos.d
$ vi mongodb-org-3.6.repo
[mongodb-org-3.6]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/3.6/x86_64/
gpgcheck=1
enabled=1
gpgkey=https://www.mongodb.org/static/pgp/server-3.6.asc
$ sudo yum install -y mongodb-org
$ service mongod start
$ ps aux | grep mongod
mongod    1267  0.4  7.8 1010008 80184 ?       Sl   11:47   0:36 /usr/bin/mongo  -f /etc/mongod.conf
```

## 安装PHP扩展
```shell_session
$ yum install openssl openssl-devel
$ sudo pecl install mongodb
$ vi /usr/local/php/lib/php.ini
extension=mongodb.so
```
