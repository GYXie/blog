title: redis快速入门
categories: Redis
tags:
  - Redis
date: 2016-09-20 00:16:03
---

## redis 简介
> Redis 是一个开源（BSD许可）的，内存中的数据结构存储系统，它可以用作数据库、缓存和消息中间件。 它支持多种类型的数据结构，如 字符串（strings）， 散列（hashes）， 列表（lists）， 集合（sets）， 有序集合（sorted sets） 与范围查询， bitmaps， hyperloglogs 和 地理空间（geospatial） 索引半径查询。 Redis 内置了 复制（replication），LUA脚本（Lua scripting）， LRU驱动事件（LRU eviction），事务（transactions） 和不同级别的 磁盘持久化（persistence）， 并通过 Redis哨兵（Sentinel）和自动 分区（Cluster）提供高可用性（high availability）。
 你可以对这些类型执行 原子操作 ， 列如： 字符串（strings）的append 命令; 散列（hashes）的hincrby命令; 列表（lists）的lpush命令; 集合（sets）计算交集sinter命令， 计算并集union命令 和 计算差集sdiff命令; 或者 在有序集合（sorted sets）里面获取成员的最高排名zrangebyscore命令.
    为了实现其卓越的性能， Redis 采用运行在 内存中的数据集工作方式. 根据您的使用情况， 您可以每隔一定时间将 数据集导出到磁盘 ， 或者 追加到命令日志中. 您也可以关闭持久化功能，将Redis作为一个高效的网络的缓存数据功能使用.
    Redis 同样支持 主从复制（能自动重连和网络断开时自动重新同步），并且第一次同步是快速的非阻塞试的同步.

其他功能包括:
- 事务（Transactions）
- 订阅分发（Pub/Sub）
- LUA脚本（Lua scripting）
- 过期自动删除key
- 内存回收
- 自动故障转移

## 常用命令
- 查看当前所有的key
```
KEYS *
```

- 查看当前redis的配置信息
```
CONFIG GET *
```
redis没有MySQL中`show dbs`的命令.Redis database的数据是固定的,在配置文件中设置.默认是16.每个database有一个数字标识,没有名字.

- reids日志位置
logfile 日志记录方式，默认值为stdout，如果设置为stdout且以守护进程方式运行，那么日志会被重定向到/dev/null,也就是不记日志。

- redis修改持久化路径和日志路径
vim redis.conf
logfile /data/redis_cache/logs/redis.log #日志路径
dir /data/redis_cache #持久化路径，修改后 记得要把dump.rdb持久化文件拷贝到/data/redis_cache下
先杀掉redis，拷贝dump.rdb，启动

- 清redis缓存
```
./redis-cli    #进入
dbsize
flushall     #执行
exit
```

- 删除redis当前数据库中的所有Key
```
flushdb
```

- 删除redis所有数据库中的key
```
flushall
```

- 建立键值对
```
set bar 1
```

- 判断键是否存在
```
exists bar
```

- 删除键
```
del bar
```

- 获得键值的数据类型
```
type foo
```
string, hash, list, set, zset

- 获得键对应的值
```
get foo
```

- 自增
```
incr foo
```

增加特定数值
```
incrby foo 3
```

类似的自减用`decr`和`decrby`
增加指定浮点数`incrbyfloat`

- 向尾部追加值
```
append bar hello
```
要追加带空格的内容，用空格引起来

- 获得字符串的长度
```
strlen bar
```

- 同时设置多个键值对
```
mset key1 value1 key2 value2 key3 value3
```
同时获得多个键值对内容
```
mget key1 key2
```

- 赋值和取值
```
hset key field value
hget key field
hmset key field1 value1 field2 value2
hmget key field1 field2
```

redis-cli连接服务器后，使用info命令查看Redis信息和状态：
```
info
```
[各个信息含义](http://my.oschina.net/tongyufu/blog/405612)

批量删除
```
redis-cli KEYS "pattern" | xargs redis-cli DEL

```

```
redis-cli -h 192.168.10.29 -p 6380de keys twenty* | xargs redis-cli -h 192.168.10.29 -p 6380de del
```

## 参考
- [redis中文](http://www.redis.cn/)  
- [redis 文档](http://www.redis.cn/documentation.html)  
- [Redis常用命令总结](http://cuiqingcai.com/431.html)  



