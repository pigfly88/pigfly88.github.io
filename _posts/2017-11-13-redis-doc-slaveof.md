---
layout: post
title:  Redis配置文件之复制
categories: nosql
---

主从复制使用slaveof将Redis实例作为另一个Redis服务器的副本。

- Redis复制是异步的，master可以配置成如果它连接的slave没有达到给定的数量，就停止接受写入。
- 如果断线较少的时间，slave可以执行部分增量复制。需要配置合理的复制积压缓冲区大小来尽可能使用增量复制。
- 复制是自动的，断线之后slave自动重连master。

slave配置：
slaveof <masterip> <masterport>

master可以设置密码：
masterauth <master-password>

当slave失去与master的连接，或者正在复制时：
1. 如果slave-serve-stale-data被设置为'yes'（默认），则slave将会仍然回复客户端的请求，可能是过时的数据，或者数据集可能只是空的，如果这是第一次同步。
2. 如果slave-serve-stale-data设置为'no'，则slave将回复"正在同步"的错误，除了INFO和SLAVEOF命令。
slave-serve-stale-data yes

slave可以配置为可写，这对于存储一些短暂数据很有用，为啥是短暂？因为与master同步后，数据会被删除，所以不建议这样做。Redis 2.6以来slave默认是只读的。
注意：slave只读不代表可以随便暴露，因为它可以执行config、debug等暴露服务器信息的命令，当然你可以对这些命令进行重命名，但这么做总是不太保险
slave-read-only yes

异步复制策略：磁盘复制和无盘复制
-------------------------------------------------------
注意：无盘复制目前处于试验阶段
-------------------------------------------------------

slave第一次连接或者重新连接master的时候，不能实现增量复制，而是全量复制，master会发送rdb给slave，有以下两种方式：
1. 磁盘复制：master创建一个子进程将rdb文件写入磁盘，然后由父进程传输给slave
2. 无盘复制：master不写磁盘，而是创建一个子进程直接通过socket发送rdb文件

使用磁盘复制，当执行bgsave生成好了rdb文件但还没开始发送的时候，其他排队等待的slave也可以拿到这个rdb文件而不必等待重新生成
如果是无盘备份，一旦传输开始，其他slave排队等待传输完毕
使用无盘复制时，可配置多长时间（秒）有多少slave才开始传输，当磁盘比较慢而网络带宽比较大的时候，无盘复制是个不错的选择
repl-diskless-sync no

如果打开无盘复制，可以配置合理的延时来等待其他slave，因为一旦开始传输，后面过来的复制请求就要排队等待，默认5秒，设置为0则不等待
repl-diskless-sync-delay 5

slave会每隔repl-ping-slave-period(默认10秒)ping一次master，如果超过repl-timeout(默认 60秒)都没有收到响应，就会认为master挂了
repl-ping-slave-period 10
repl-timeout 60

我们可以控制在主从同步时是否禁用TCP_NODELAY。如果是yes，那么master会使用更少的TCP包和更少的带宽来向slave传输数据。
但是这可能会增加一些同步的延迟，大概会达到40毫秒左右。如果是no，那么数据同步的延迟时间会降低，但是会消耗更多的带宽。
repl-disable-tcp-nodelay no

缓冲积压队列，redis会把最近的命令放到队列里，供slave进行增量复制，设置得越大越有机会实现增量复制而非全量复制
repl-backlog-size 1mb

超过多长时间没有slave请求复制，缓冲积压队列将被释放
repl-backlog-ttl 3600

当master挂了，Redis Sentinel通过slave-priority来决定哪个slave接管成为master，最小的最优先，0代表永远不接管
slave-priority 100

配置master在M秒内有N个slave连接才可写，可以把其中一个值设置成0来关闭此功能，比如说10秒内有3台slave连接master才可写
min-slaves-to-write 3
min-slaves-max-lag 10

master通过info获取slave的ip地址和端口，当使用了端口转发或NAT的时候，需要配置IP地址映射
slave-announce-ip 5.5.5.5
slave-announce-port 1234