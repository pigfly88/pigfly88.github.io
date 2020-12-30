---
layout: post
category: "nosql"
title:  "Redis深度历险学习笔记"
---

Redis应用场景：缓存、分布式锁、消息队列、限流

Redis支持的数据结构：string, hash, list, set, zset, bitmap, hyperloglog, geo, pub/sub, 布隆过滤器

list是用快速链表(linkedlist + ziplist)实现的，插入和删除的速度非常快(push, pop)，时间复杂度是0(1)，但索引定位比较慢(lindex, ltrim)，时间复杂度为O(n)

hash底层使用数组+链表实现的，第一维 hash 的数组位置碰撞时，就会将碰撞的元素使用链表串接起来

set底层是用字典实现的，字典中所有的 value 都是 NULL

zset类似于 Java 的 SortedSet 和 HashMap 的结合体，一方面它是一个 set，保证了内部 value 的唯一性，另一方面它可以给每个 value 赋予一个 score，代表这个 value 的排序权重。它的内部实现用的是一种叫做**跳跃列表**的数据结构。

### 千帆竞发-分布式锁

```shell
> set lock:codehole true ex 5 nx
OK
... do something ...
> del lock:codehole
```

Redis 的分布式锁不能解决超时问题，如果在加锁和释放锁之间的逻辑执行的太长，以至于超出了锁的超时限制，就会出现问题。

因为这时候第一个线程持有的锁过期了，临界区的逻辑还没有执行完，这个时候第二个线程就提前重新持有了这把锁，导致临界区代码不能得到严格的串行执行。这个时候第二个线程有可能会把第一个线程持有的锁释放掉，解决方案是将value值设置为一个随机数，释放锁的时候先判断value值是否相等，相等才删除。

```shell
tag = random.nextint()  # 随机数
if redis.set(key, tag, nx=True, ex=5):
    do_something()
    redis.delifequals(key, tag)  # 假想的 delifequals 指令，使用Lua脚本保证原子性
```
	
```lua
# delifequals
if redis.call("get",KEYS[1]) == ARGV[1] then
    return redis.call("del",KEYS[1])
else
    return 0
end
```

### 缓兵之计-延时队列

#### 异步消息队列

生产者rpush，消费者lpop。

#### 队列空了怎么办？

如果队列空了，客户端就会陷入 pop 的死循环，空轮询不但拉高了客户端的 CPU，redis 的 QPS 也会被拉高，如果这样空轮询的客户端有几十来个，Redis 的慢查询可能会显著增多。在程序里面sleep可以解决，但是没办法对时间做精准控制，会导致消息的延迟增大。Redis 提供了**blpop/brpop**实现了队列为空时的阻塞读，阻塞读在队列没有数据的时候，会立即进入休眠状态，一旦数据到来，则立刻醒过来。消息的延迟几乎为零。

注意阻塞读可能会导致客户端的空闲连接超时自动断开，这时候要做好异常捕获进行重试。


#### 更安全的队列

如果消息在发送过程中丢失，或者客户端程序在pop之后崩溃了，这时候消息就会丢失。Redis提供了rpoplpush，这条命令从第一个队列取出一个消息并放到另外一个队列中，当消息处理完以后，再把消息从另外一个队列中移除。

另外，可以添加一个客户端来监控另外一个队列，如果有某些消息已经在这个列表中存在很长时间了（即超过一定的处理时限），就把这些超时消息重新加入到第一个队列中重做。


#### 延时队列

当客户端获取分布式锁失败的时候可以把这个请求放到延时队列里面。

延时队列可以通过 zset 来实现。我们把消息内容作为key，时间戳作为score，用zadd来生产消息，然后用多个线程调用zrangebyscore指令获取N秒之前的数据轮询进行处理，多个线程是为了保障可用性，万一挂了一个线程还有其它线程可以继续处理。因为有多个线程，所以需要考虑并发争抢任务，确保任务不能被多次执行。

```python
def delay(msg):
    msg.id = str(uuid.uuid4())  # 保证 value 值唯一
    value = json.dumps(msg)
    retry_ts = time.time() + 5  # 5 秒后重试
    redis.zadd("delay-queue", retry_ts, value)


def loop():
    while True:
        # 最多取 1 条
        values = redis.zrangebyscore("delay-queue", 0, time.time(), start=0, num=1)
        if not values:
            time.sleep(1)  # 延时队列空的，休息 1s
            continue
        value = values[0]  # 拿第一条，也只有一条
        success = redis.zrem("delay-queue", value)  # 从消息队列中移除该消息
        if success:  # 因为有多进程并发的可能，最终只会有一个进程可以抢到消息
            msg = json.loads(value)
            handle_msg(msg)
```

上面的算法中同一个任务可能会被多个进程取到之后再使用 zrem 进行争抢，那些没抢到的进程都是白取了一次任务，这是浪费。可以考虑使用 lua scripting 来优化一下这个逻辑，将 zrangebyscore 和 zrem 一同挪到服务器端进行原子化操作，这样多个进程之间争抢任务时就不会出现这种浪费了。

```lua
local value = redis.call("zrangebyscore", "delay-queue", 0, ARGV[1], "LIMIT", "0", "1")
if not value or not value[1] then
 return nil
end
redis.log(redis.LOG_DEBUG, cjson.encode(value))
redis.call("zrem", "delay-queue", value[1])
return value[1]
```

### 节衣缩食-位图


#### bit,byte和字符的关系

bit: 位，是计算机中最小的数据单位（0或1）

byte: 字节，基本计量单位，1字节=8位，1字节可以存储1个英文字母或者半个汉字

```shell
> set test1 'h'

OK

> bitfield test1 get u8 0

1) (integer) 104 # h的ASCII码就是104，转换成二进制就是1101000，这就是字符h在计算机内部存储的内容
```

#### 签到统计

```shell
# 签到
> setbit signin:202008:1 0 1 # 2020年8月份uid=1的用户在1号签到
> setbit signin:202008:1 1 1 # 2020年8月份uid=1的用户在2号签到
> setbit signin:202008:1 2 1
> setbit signin:202008:1 4 1
> setbit signin:202008:1 5 1

> setbit signin:202008:1 7 1
> setbit signin:202008:1 9 1

# 检查2号是否签到
> getbit signin:202008:1 1

# 统计8月份的签到次数
> bitcount signin:202008:1
(integer) 7

# 获取8月份31天的签到情况
# 1990197248通过decbin转成二进制是：1110110101000000000000000000000
> bitfield signin:202008:1 get u31 0
1) (integer) 1990197248

# 获取首次签到的日期
> bitpos signin:202008:1 1

# 第一周的签到情况
# 从第0个位开始取7个位（一周是7天），结果是无符号数(u)
> bitfield signin:202008:1 u7 0
1) (integer) 118 # 118转成二进制是：1110110

# 第二周的签到情况
> bitfield signin:202008:1 get u7 6
1) (integer) 40 # 40转成二进制是：101000

# 8月份前8天的签到次数
# 为啥是8天？因为bitcount的起始位置参数的单位是字符，1个字符是8bit
> bitcount signin:202008:1 0 0
(integer) 7

# 那么第一周的签到天数就是上面的结果减去第8天的
# 获取第8天的签到情况为1（已签到）,那么第一周的签到天数就是7-1=6
> getbit signin:202008:1 7
(integer) 1
```

### 四两拨千斤-HyperLogLog

应用场景：统计网站页面的UV，UV需要去重，如果用set太占空间了，Redis 提供了 HyperLogLog 数据结构就是用来解决这种统计问题的。HyperLogLog 提供**不精确的去重计数**方案。
```
> pfadd uv:page1 user1
> pfadd uv:page1 user2
> pfcount uv:page1
(integer) 2
```

### 层峦叠嶂-布隆过滤器

当布隆过滤器说某个值存在时，这个值可能不存在；**当它说不存在时，那就肯定不存在。**

使用场景：内容推荐系统去重、爬虫URL去重、垃圾邮箱过滤、防止缓存穿透

### 简单限流

例子：每分钟限制
```
incr act:uid:minute # 缺点是分钟的界限是自然分钟，非连续的可滑动的。
```

```
key = act:uid
zadd microtime microtime
zremrangebyscore key 0 microtime-60 # 移除60秒之前的数据
zcard key
expire key 60
```

### 漏斗限流

```
cl.throttle laoqian:reply 15 30 60 1
```

上面这个指令的意思是允许「用户老钱回复行为」的频率为每 60s 最多 30 次(漏水速率)，漏斗的初始容量为 15，也就是说一开始可以连续回复 15 个帖子，然后才开始受漏水速率的影响。

### 持久化

rdb是全量数据的快照，是bgsave生成的二进制文件，aof 日志记录了对内存进行修改的每条指令，用来做增量持久化。因为bgsave会耗费较长时间，不够实时，在停机的时候会导致大量丢失数据，所以需要aof来配合使用。在redis实例重启时，优先使用aof来恢复内存的状态，如果没有aof日志，就会使用rdb文件来恢复。

bgsave的原理：**fork and cow**。fork是指redis通过创建子进程来进行bgsave操作，cow指的是copy on write，子进程创建后，父子进程共享数据段，父进程继续提供读写服务，写脏的页面数据会逐渐和子进程分离开来。

```py
pid = os.fork()
if pid > 0:
    handle_client_requests()  # 父进程继续处理客户端请求
if pid == 0:
    handle_snapshot_write()  # 子进程处理快照写磁盘
if pid < 0:
    # fork error
```

bgsave自动触发策略配置：

```
# save <seconds> <changes>
save 900 1    # 900秒内有一次更新操作
save 300 10   # 300秒内有10次更新操作
save 60 10000 # 60秒内有1w次更新操作
```

aof把每次操作的命令记录下来，所以文件会越来越大，Redis会定期做aof重写，或者手动执行bgrewriteaof命令，压缩aof文件日志大小。Redis4.0之后有了**混合持久化**的功能，将bgsave的全量和aof的增量这两种方式结合起来，这样既保证了恢复的效率又兼顾了数据的安全性。

对于使用AOF持久化方式的业务，仍然推荐打开RDB快照功能，因为快照文件对于主从同步、数据库备份、将数据发送到远程服务器进行备份或者发生重大事故需要回滚数据等场景都有重要意义。

RDB文件的加载速度一般是每载入1G数据需要10~20秒

AOF 日志内容首先是写到了内核为文件描述符分配的一个**内存缓存**中，然后内核会异步将脏数据刷回到磁盘的。如果机器突然宕机，AOF 日志可能还没来得及完全刷到磁盘中，数据就会丢失，Redis 提供了如下刷盘时机供我们选择：
```
# appendfsync always # 每次更改操作都刷盘
appendfsync everysec # 默认，每秒刷盘一次，最多丢失1~2秒的数据
# appendfsync no # 让操作系统决定何时刷盘
```

#### 主从架构下的优化

1. 快照是通过开启子进程的方式进行的，它是一个比较耗资源的操作：遍历整个内存，大块写磁盘会加重系统负载
1. AOF 的 fsync 是一个耗时的 IO 操作，它会降低 Redis 性能，同时也会增加系统 IO 负担

所以通常 Redis 的主节点是不会进行持久化操作的，持久化操作主要在从节点进行。从节点是备份节点，没有来自客户端请求的压力，它的操作系统资源往往比较充沛。可以按以下策略来调整优化：

1. 主库关闭AOF和快照，允许从库访问即可。写个定时脚本每晚12点手动做一次bgsave快照，并将快照文件通过 SCP 备份到远程节点上
1. 从库开启AOF everysec，关闭快照。写个定时脚本每晚12点对AOF文件打包备份(tar)，然后执行bgrewriteaof重写AOF

需要注意的是要保障主从同步通畅，如果从节点长期连不上主节点，就会出现主从数据不一致的问题，会造成数据丢失，需要做好报警机制。

#### pipeline

客户端使用pipeline机制可以一次向服务端发送N条命令。然后等待这N个结果一起返回，将多次IO往返的时间缩减为一次。

#### keys 和 scan

如果要找出含有固定前缀的10w个key，如果是用keys的话，因为redis是单线程的，会阻塞其他的读写操作。scan指令可以无阻塞的提取出指定模式的key列表，但是会有一定的重复概率，在客户端做一次去重就可以了，但是整体所花费的时间会比直接用keys指令长。

### 有备无患-主从同步

#### CAP 原理

- C - Consistent ，一致性
- A - Availability ，可用性
- P - Partition tolerance ，分区容忍性

分布式系统的节点往往都是分布在不同的机器上进行网络隔离开的，这意味着必然会有网络断开的风险，这个网络断开的场景的专业词汇叫**网络分区**。

一句话概括 CAP 原理就是：**网络分区发生时，一致性和可用性两难全**。

Redis 保证**最终一致性**，从节点会努力追赶主节点。

#### 同步机制

Redis可以使用主从同步，从从同步。快照同步是一个非常耗费资源的操作，第一次同步时，它首先需要在主库上进行一次 bgsave 将当前内存的数据全部快照到磁盘文件中，然后再将快照文件的内容全部传送到从节点。从节点将快照文件接受完毕后，立即执行一次全量加载，加载之前先要将当前内存的数据清空。加载完毕后通知主节点继续进行增量同步。

### 李代桃僵-Sentinel

sentinel是redis高可用的解决方案，sentinel系统可以监视一个或者多个master服务，当某个master服务下线时，自动将该master下的某个slave节点升级为master，其他slave节点自动指向新的master同步数据。

sentinel一般会部署多个节点以防止sentinel本身出现单点故障，客户端先连接sentinel获得主节点地址，然后再去连接主节点进行数据交互。

![](/images/redis-sentinel.webp)

```
docker build .
docker run -d --name redis-sentinel-getstart -v ${PWD}/conf:/etc/redis redis:4.0
docker exec -it redis-sentinel-getstart bash
nohup redis-server --port 6380 --slaveof 127.0.0.1 6379 &
nohup redis-server --port 6381 --slaveof 127.0.0.1 6379 &

nohup redis-server /etc/redis/sentinel-26379.conf --sentinel &
nohup redis-server /etc/redis/sentinel-26380.conf --sentinel &
nohup redis-server /etc/redis/sentinel-26381.conf --sentinel &

redis-cli -p 26379 sentinel get-master-addr-by-name mymaster
```

```php
$sentinel = new RedisSentinel('127.0.0.1', 26379, 2.5); // 2.5 sec timeout.
$sentinel->getMasterAddrByName('mymaster'); // Return the ip and port number of the master with that name.
$sentinel->sentinels('mymaster'); // Return a list of sentinel instances for this master, and their state.
```

Redis Cluster着眼于扩展性，在单个redis内存不足时，使用Cluster进行分片存储。

未完待续。。。

## 参考资料
1. [Redis 深度历险：核心原理与应用实践](https://juejin.im/book/6844733724618129422)