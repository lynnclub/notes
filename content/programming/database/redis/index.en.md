---
title: "Redis"
type: "docs"
weight: 1
---

## Data Structures

1. string: Simple Dynamic String (SDS), self-created data structure. Compared to C language strings, it adds length and remaining records to accelerate statistics and prevent overflow. Both key and value are SDS, hash mapping in dictionary, key saved after hashing, hash duplicates expand unidirectional linked list.
2. list: Doubly linked list.
3. hash: Hash.
4. sorted set: Skip list, achieves fast node access by maintaining multiple pointers to other nodes in each node. Solves the problem of difficulty finding specific values in ordered linked list structures.

Problems

1. Cache avalanche: Large amounts of data expire simultaneously (use random expiration times);
2. Cache breakdown: ?
3. Cache penetration: Don't cache if not found (cache null objects, data preheating and degradation, active cache updates);
4. High availability solutions: Master-slave replication, Cluster clustering.
5. Hot key: Hot keys cause performance bottlenecks in certain instances within cluster, making entire cluster bottlenecked. Can use cluster sharding mechanism, add prefixes to scatter keys across N instances.
6. Big key: Maximum value 512MB. After big key sharding, memory usage of certain instance far exceeds other instances, causing insufficient memory and potentially dragging down entire cluster. Also, because Redis is single-threaded, operations like hgetall, hmget, lrange have large I/O throughput and may block cluster. Can split big keys.

## Reclaim Strategy

![gc](gc.png)

Generally adopts lazy deletion + periodic deletion strategy.

keys and scan can retrieve expired but not yet reclaimed keys, can be used for data recovery. keys may cause service lag, not recommended.

Difference between delete and expire

delete: Direct deletion
expires: Wait for reclamation

## Hash Collision and Rehash

Redis uses hash table to save all key-value pairs, adopts chained hashing to resolve conflicts, using linked list to save multiple conflicting elements in hash bucket.

When conflict chain becomes too long, affects efficiency, will rehash to increase number of hash buckets, reducing elements per bucket.

Time Complexity
[https://blog.csdn.net/qq_23564667/article/details/110917900](https://blog.csdn.net/qq_23564667/article/details/110917900)

## Server

### Service Start/Stop

```shell
# Start
redis-server /path/to/redis.conf
# Stop
redis-cli -p 7000 -a 123456 shutdown

# CentOS
systemctl start redisd
systemctl stop redisd

# Brew
brew services start redis
brew services stop redis
```

### Create Cluster

After starting redis servers with different ports, execute creation command below

```shell
# No replicas, requires at least 3 nodes
redis-cli --cluster create 127.0.0.1:7000 127.0.0.1:7001 127.0.0.1:7002

# Master-slave replication, 3 masters 3 slaves
redis-cli --cluster create  --cluster-replicas 1 127.0.0.1:7000 127.0.0.1:7001 127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005
```

--cluster-replicas number of replicas

### Create Cluster Docker

```shell
for port in `seq 7000 7002`; do \
  mkdir -p ${port}/conf \
  && PORT=${port} envsubst < redis-cluster.tmpl > ${port}/conf/redis.conf \
  && mkdir -p ${port}/data; \
done
```

```text
#Node port, external communication port
port ${PORT}
#requirepass 123456
#masterauth 123456
protected-mode no
daemonize no
#Persistence mode
appendonly yes
#Whether to enable cluster
cluster-enabled yes
#Cluster config file
cluster-config-file nodes.conf
#Connection timeout
cluster-node-timeout 5000
#Cluster node IPs, need to fill host machine ip
cluster-announce-ip host.docker.internal
#Cluster node mapping port
cluster-announce-port ${PORT}
#Cluster bus port
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

## Connect to Server

```shell
redis-cli -h 127.0.0.1 -p 6379 -c -n 0 -a 123456
```

1. h server address
2. p port
3. n database, equivalent to select command
4. a password
5. c enable cluster mode, redis cluster

## scan and unlink

```
redis-cli -h 127.0.0.1 -p 6379 -n 0 -a 123456 --scan --pattern test*w | xargs redis-cli -h 127.0.0.1 -p 6379 -n 0 -a 123456 unlink
```

1. scan: Cursor-based step scanning of keys, similar effect to keys command but doesn't block server
2. unlink: Asynchronous deletion, non-blocking, requires redis version 4+

## Bloom Filter

When element is added to set, maps element to multiple bits in bit array through multiple hash functions and sets these bits to 1. During lookup, maps element to be searched to multiple bits in bit array through same hash functions and checks if all bits are 1. If all are 1, element is likely in set; if not all 1, element definitely not in set.

Bloom filter is probabilistic data structure. Advantage is space efficiency and query time far exceed general algorithms. Disadvantage is certain false positive rate and difficulty in deletion.

## High Availability Solutions

Master-slave mode (master, slave), Sentinel mode, Cluster mode.

### Sentinel Mode

Sentinel mode is high-availability redis deployment solution. Sentinel monitors status of redis master and slave nodes. When master node fails, sentinel automatically promotes certain slave node to master and switches other slave nodes to new master.

### Redis and MySQL Data Consistency

Update database first, then update cache
Update cache first, then update database

Both solutions have concurrency problems. When two requests concurrently update same data, may cause inconsistent data between cache and database.

Delete cache first, then update database
Update database first, then delete cache

Theoretically, updating database first then deleting cache also has data inconsistency problems, but in practice this problem probability is not high.

So, "update database first + then delete cache" solution can ensure data consistency. Can also set shorter expiration time as fallback.

However, cache deletion may fail, can improve success rate through retry mechanism.
