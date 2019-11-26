---
layout: post
category: "mix"
title:  "ZooKeeper"
---

ZooKeeper是一种**分布式协调服务**，可以在分布式系统中共享配置，协调锁资源，提供命名服务。

## 数据结构

ZooKeeper的数据结构很像数据结构里面的树，也很像文件系统的目录。
![](/images/zookeeper-data-struct.webp)

Zookeeper里的节点叫做Znode，每个节点包含这些信息：
- data：Znode存储的数据信息。
- ACL：记录Znode的访问权限，即哪些人或哪些IP可以访问本节点。
- stat：包含Znode的各种元数据，比如事务ID、版本号、时间戳、大小等等。
- child：当前节点的子节点引用，类似于二叉树的左孩子右孩子。

![](/images/zookeeper-znode.webp)

## 应用场景

- 分布式锁
- 服务注册与发现
- 共享配置和状态信息