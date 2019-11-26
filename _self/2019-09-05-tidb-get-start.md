---
layout: post
category: "mysql"
title:  "TiDB上手"
---

## 数据存储

内存存储：TiKV，采用 Key-Value 来保存数据，可以将 TiKV 看作一个巨大的 Map，其中 Key 和 Value 都是原始的 Byte 数组，Key 按照 Byte 数组总的原始二进制比特位排序。

持久化：RocksDB，TiKV 的数据落地交给 RocksDB。

解决单点故障：通过 Raft 协议，把数据复制到其他机器上。

![](/images/tikv-data-store.png)

通过单机的 RocksDB 将内存数据快速落地到磁盘上（先通过 Raft 再写入），然后通过 Raft 协议 把数据复制到其他机器上，防止单点故障。