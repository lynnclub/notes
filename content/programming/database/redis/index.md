---
title: "Redis"
type: "docs"
weight: 1
---

## 数据结构

1. string：简单动态字符串 SDS，自创数据结构，相对 C 语言字符串多了长度和剩余记录，加速统计、防止溢出。key 和 value 都是 SDS，hash 映射在字典里，key 经 hash 保存，hash 重复拓展单向链表。
2. list：双向链表。
3. hash：哈希。
4. sorted set：跳跃表 skiplist，通过在每个节点中维持多个指向其他节点的指针，从而达到快速访问节点的目的。解决了有序链表结构查找特定值困难的问题。

问题

1. 缓存雪崩，大量数据同时过期（随机过期时间）；
2. 缓存击穿，？
3. 缓存穿透，查不到就不缓存（缓存空对象、数据预热与降级、主动更新缓存）；
4. 高可用方案：主从复制、Cluster 集群。
5. hot key：热 key 导致集群内某个单例出现性能瓶颈，从而使整个集群出现性能瓶颈。可利用集群分片机制，加前缀打散 key 使其落在 N 个单例中。
6. big key：值最大 512MB。大 Key 分片后，某个实例内存用量远大于其他实例，导致内存不足，可能会拖累整个集群。而且，因为 Redis 单线程，hgetall、hmget、lrange 等操作 IO 吞吐量大，存在阻塞集群的可能性。可拆分大 key。

## 回收策略

![gc](gc.png)

一般采用 惰性删除 + 定期删除 的策略。

keys 和 scan 可以获取到已过期未回收的 key，可以用来恢复数据。keys 可能会造成服务卡顿，不推荐使用。

delete 和 expire 区别

delete 直接删除  
expires 等待回收

## hash 冲突和 rehash

Redis 使用了一个 hash 表来保存所有键值对，采用链式 hash 的方式解决冲突，对 hash 桶中冲突的多个元素使用一个链表保存。

当冲突链过长，会影响效率，会 rehash 增加 hash 桶的数量，减少单桶中的元素数量。

时间复杂度  
[https://blog.csdn.net/qq_23564667/article/details/110917900](https://blog.csdn.net/qq_23564667/article/details/110917900)

## 服务端

### 服务启停

```shell
# 启动
redis-server /path/to/redis.conf
# 关停
redis-cli -p 7000 -a 123456 shutdown

# CentOS
systemctl start redisd
systemctl stop redisd

# Brew
brew services start redis
brew services stop redis
```

### 创建集群

启动端口不同的 redis 服务器后，执行下述创建命令

```shell
# 无副本，至少需要3个节点
redis-cli --cluster create 127.0.0.1:7000 127.0.0.1:7001 127.0.0.1:7002

# 主从复制，主3从3
redis-cli --cluster create  --cluster-replicas 1 127.0.0.1:7000 127.0.0.1:7001 127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005
```

–cluster-replicas 副本数量

### 创建集群 Docker

```shell
for port in `seq 7000 7002`; do \
  mkdir -p ${port}/conf \
  && PORT=${port} envsubst < redis-cluster.tmpl > ${port}/conf/redis.conf \
  && mkdir -p ${port}/data; \
done
```

```text
#节点端口，即对外提供通信的端口
port ${PORT}
#requirepass 123456
#masterauth 123456
protected-mode no
daemonize no
#持久化模式
appendonly yes
#是否启用集群
cluster-enabled yes
#集群配置文件
cluster-config-file nodes.conf
#连接超时时间
cluster-node-timeout 5000
#集群各节点IP地址，需要填写宿主机ip
cluster-announce-ip host.docker.internal
#集群节点映射端口
cluster-announce-port ${PORT}
#集群总线端口
cluster-announce-bus-port 1${PORT}
```

```text
127.0.0.1 host.docker.internal
```

```yaml
version: "3"

x-image: &default-image redis:alpine
x-restart: &default-restart always

services:
  node7000:
    image: *default-image
    restart: *default-restart
    volumes:
      - ./7000/conf:/etc/redis
      - ./7000/data:/data
    ports:
      - "7000:7000"
    command: redis-server /etc/redis/redis.conf
  node7001:
    image: *default-image
    restart: *default-restart
    volumes:
      - ./7001/conf:/etc/redis
      - ./7001/data:/data
    ports:
      - "7001:7001"
    command: redis-server /etc/redis/redis.conf
    depends_on:
      - node7000
  node7002:
    image: *default-image
    restart: *default-restart
    volumes:
      - ./7002/conf:/etc/redis
      - ./7002/data:/data
    ports:
      - "7002:7002"
    command: redis-server /etc/redis/redis.conf
    depends_on:
      - node7000
```

## 连接服务器

```shell
redis-cli -h 127.0.0.1 -p 6379 -c -n 0 -a 123456
```

1. h 服务器地址
2. p 端口
3. n 库，等同 select 命令
4. a 密码
5. c 开启集群模式，redis cluster

## scan 与 unlink

```
redis-cli -h 127.0.0.1 -p 6379 -n 0 -a 123456 --scan --pattern test*w | xargs redis-cli -h 127.0.0.1 -p 6379 -n 0 -a 123456 unlink
```

1. scan 游标分步扫描 key，效果类似 keys 命令，但不会阻塞服务器
2. unlink 异步删除，不阻塞，需要 redis 版本 4+

## 布隆过滤器

当一个元素被加入集合时，通过多个哈希函数将该元素映射成一个位数组中的多个位，并将这些位都置为 1。在查找时，通过同样的哈希函数将待查元素映射到位数组中的多个位，并检查这些位是否都为 1。如果都为 1，则说明待查元素很可能在集合中；如果不全为 1，则说明待查元素肯定不在集合中。

布隆过滤器是一种概率型数据结构，优点是空间效率和查询时间都远远超过一般的算法，缺点是有一定的误识别率和删除困难。

## 高可用方案

主从模式（master、slave），哨兵模式（Sentinel），集群模式（Cluster）。

### 哨兵模式

哨兵模式是一个高可用的 redis 部署方案。哨兵负责监控 redis 主节点和从节点的状态，当主节点宕机时，哨兵会自动将某个从节点升级为主节点，并将其他从节点切换到新主节点。

### redis 和 mysql 数据一致性

先更新数据库，再更新缓存  
先更新缓存，再更新数据库

这两个方案都存在并发问题，当两个请求并发更新同一条数据的时候，可能会出现缓存和数据库中的数据不一致的现象。

先删除缓存，再更新数据库  
先更新数据库，再删除缓存

理论上分析，先更新数据库，再删除缓存也是会出现数据不一致性的问题，但是在实际中，这个问题出现的概率并不高。

所以，「先更新数据库 + 再删除缓存」的方案，是可以保证数据一致性的。还可以设置较短的过期时间兜底。

但是，删除缓存可能会失败，可以通过重试机制提高成功率。
