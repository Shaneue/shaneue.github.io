---
title: Redis
date: 2018-11-18 16:49:45
updated: 2020-08-10 12:00:00
tags: [Caching]
typora-root-url: ../
---

An open source, in-memory data structure store, used as a database, cache and message broker

<!-- more -->

## Usage Scenario

Most commonly used as in-memory cache. 

## Data Types

二进制安全的字符串，会有一个length属性，而不是以'\0'判断结尾。

数据结构源码文件：sds、dict、intset、ziplist、quicklist、hyperloglog、zipmap（已经废弃，代码中还留着为了兼容旧的rdb）、t_**(目前有6个文件)

quicklist是list结构的底层实现，将ziplist串起来了，替换掉了旧的linkedlist。

The maximum allowed key size is 512 MB.

通过命令可以操作的常用数据类型：

> - Binary-safe strings.
> - Lists: collections of string elements sorted according to the order of insertion. They are basically linked lists.
> - Sets: collections of unique, unsorted string elements.
> - Sorted sets, similar to Sets but where every string element is associated to a floating number value, called score. The elements are always taken sorted by their score, so unlike Sets it is possible to retrieve a range of elements (for example you may ask: give me the top 10, or the bottom 10).（skiplist and hashtable）
> - Hashes, which are maps composed of fields associated with values. Both the field and the value are strings. This is very similar to Ruby or Python hashes.
> - Bit arrays (or simply bitmaps): it is possible, using special commands, to handle String values like an array of bits: you can set and clear individual bits, count all the bits set to 1, find the first set or unset bit, and so forth.
> - HyperLogLogs: this is a probabilistic data structure which is used in order to estimate the cardinality of a set. Don't be scared, it is simpler than it seems... See later in the HyperLogLog section of this tutorial.
> - Streams: append-only collections of map-like entries that provide an abstract log data type. They are covered in depth in the Introduction to Redis Streams.

操作集合，集合为空时的基本规则：

> - When we add an element to an aggregate data type, if the target key does not exist, an empty aggregate data type is created before adding the element.
> - When we remove elements from an aggregate data type, if the value remains empty, the key is automatically destroyed. The Stream data type is the only exception to this rule.
> - Calling a read-only command such as LLEN (which returns the length of the list), or a write command removing elements, with an empty key, always produces the same result as if the key is holding an empty aggregate type of the type the command expects to find.

Redis不允许嵌套数据类型。

### Commands

mset、mget

type

expire、persist、pexpire、ttl、pttl

lpush、rpush、lpop、rpop、lrange、ltrim、blpop、brpop、rpoplpush、brpoplpush

hset、hmset、hget、hmget

sadd、smembers、sismember、sinter、sunion、sunionstore、sdiff

zadd、zrange、zrevrange、zrange with scores、zrangebyscore、zremrangebyscore、zrank

setbit、getbit、bitcount

pfadd、pfcount

## Expires and Eviction

> Since Redis 2.6 the expire error is from 0 to 1 milliseconds.
>
> For expires to work well, the computer time must be taken stable. If you move an RDB file from two computers with a big desync in their clocks, funny things may happen (like all the keys loaded to be expired at loading time).
>
> Even running instances will always check the computer clock, so for instance if you set a key with a time to live of 1000 seconds, and then set your computer time 2000 seconds in the future, the key will be expired immediately, instead of lasting for 1000 seconds.

### How Redis expires keys

6.0做了较大改进，stale keys比例默认降到10%，并可配置

> A key is passively expired simply when some client tries to access it, and the key is found to be timed out.
>
> Periodically Redis tests a few keys at random among keys with an expire set. All the keys that are already expired are deleted from the key space.
>
> Specifically this is what Redis does 10 times per second:
>
> 1. Test 20 random keys from the set of keys with an associated expire. 
> 2. Delete all the keys found expired.
> 3. If more than 10% of keys were expired, start again from step 1.
>
> In order to obtain a correct behavior without sacrificing consistency, when a key expires, a [DEL](https://redis.io/commands/del) operation is synthesized in both the AOF file.
>
> The default effort of the expire cycle will try to avoid having more than ten percent of expired keys still in memory, and will try to avoid consuming more than 25% of cpu usage and to add latency to the system.

源码在expire.c

### Eviction Policies

在设置了maxmemory之后，可以设置maxmemory-policy，6.0中有8种：

> - volatile-lru -> Evict using approximated LRU, only keys with an expire set.
> - allkeys-lru -> Evict any key using approximated LRU.
> - volatile-lfu -> Evict using approximated LFU, only keys with an expire set.
> - allkeys-lfu -> Evict any key using approximated LFU.
> - volatile-random -> Remove a random key having an expire set.
> - allkeys-random -> Remove a random key, any key.
> - volatile-ttl -> Remove the key with the nearest expire time (minor TTL)
> - noeviction -> Don't evict anything, just return an error on write operations.（default）

maxmemory-samples 5（default）

### How the eviction process works

> It is important to understand that the eviction process works like this:
>
> - A client runs a new command, resulting in more data added.
> - Redis checks the memory usage, and if it is greater than the `maxmemory` limit , it evicts keys according to the policy.
> - A new command is executed, and so forth.
>
> So we continuously cross the boundaries of the memory limit, by going over it, and then by evicting keys to return back under the limits.
>
> If a command results in a lot of memory being used (like a big set intersection stored into a new key) for some time the memory limit can be surpassed by a noticeable amount.

Redis LRU algorithm is not an exact implementation.

这里需要用到maxmemory-samples配置。

## Persistence

默认开启rdb

> save 900 1
>
> save 300 10
>
> save 60 10000

### How RDB works

> Whenever Redis needs to dump the dataset to disk, this is what happens:
>
> - Redis forks. We now have a child and a parent process.
>
> - The child starts to write the dataset to a temporary RDB file.
>
> - When the child is done writing the new RDB file, it replaces the old one.
>
>
> This method allows Redis to benefit from copy-on-write semantics.

### Append-only File

AOF支持重写压缩。

有三种配置选项：

- appendfsync always: fsync every time a new command is appended to the AOF. Very very slow, very safe.
- appendfsync everysec: fsync every second. Fast enough (in 2.4 likely to be as fast as snapshotting), and you can lose 1 second of data if there is a disaster.
- appendfsync no: Never fsync, just put your data in the hands of the Operating System. The faster and less safe method. Normally Linux will flush data every 30 seconds with this configuration, but it's up to the kernel exact tuning.

### Conclusion

可以将持久化特性关闭，将配置里的save **注释掉即可。

两种方式都开启的话，会选择使用AOF来重建数据库。

## Memory Optimization

> hash-max-ziplist-entries 512
>
> hash-max-ziplist-value 64
>
> zset-max-ziplist-entries 128
>
> zset-max-ziplist-value 64
>
> set-max-intset-entries 512

Redis消耗的Resident Set Size更接近于peak memory usage。最好配置maxmemory，防止内存被使用光。

maxmemory对应的是used memory。

## Replication

> Synchronous replication of certain data can be requested by the clients using the WAIT command. However WAIT is only able to ensure that there are the specified number of acknowledged copies in the other Redis instances, it does not turn a set of Redis instances into a CP system with strong consistency: acknowledged writes can still be lost during a failover, depending on the exact configuration of the Redis persistence.

将fsync配置成always同时配合WAIT命令是可以实现Strong Consistency的，但是这么做会完全牺牲可用性，因为任何一个节点fail，都会使集群不接受writes。

只需要配置replicaof <masterip> <masterport>标识进程是一个replica。

> This system works using three main mechanisms:
>
> - When a master and a replica instances are well-connected, the master keeps the replica updated by sending a stream of commands to the replica, in order to replicate the effects on the dataset happening in the master side due to: client writes, keys expired or evicted, any other action changing the master dataset.
> - When the link between the master and the replica breaks, for network issues or because a timeout is sensed in the master or the replica, the replica reconnects and attempts to proceed with a partial resynchronization: it means that it will try to just obtain the part of the stream of commands it missed during the disconnection.
> - When a partial resynchronization is not possible, the replica will ask for a full resynchronization. This will involve a more complex process in which the master needs to create a snapshot of all its data, send it to the replica, and then continue sending the stream of commands as the dataset changes.

> Redis uses by default asynchronous replication.
>
> Masters with persistence turned off configured to auto restart are dangerous.
>
> Every Redis master has a replication ID: it is a large pseudo random string that marks a given history of the dataset.
>
> If the replica is referring to an history (replication ID) which is no longer known, than a full resynchronization happens: in this case the replica will get a full copy of the dataset, from scratch.
>
> Instances have actually two replication IDs the main ID and the secondary ID.
>
> Redis version 2.8.18 is the first version to have support for diskless replication.
>
> Writable replicas before version 4.0 were incapable of expiring keys with a time to live set.
>
> Computing slow Set or Sorted set operations and storing them into local keys is an use case for writable replicas that was observed multiple times.

### High Availability

在无数据分片的情况下，可使用redis-sentinel来监控节点运行状况，在master节点宕机时，升级slave为master，并通知客户端进行切换。

最简易的方案至少需要3个节点的sentinel，并设置quorum为2，因为必须至少过半的投票节点同意才能解决网络分区的问题。

在搭建时，启动Replica节点，连接上master，3个sentinel节点只需要监控master节点即可，它们会自动获取所有节点的信息。

正常情况下，1G内存能支持一千万个value值很小的key。

简单部署方式：

![](/images/redis1.png)

### Cluster

至少需要6个节点，3个用于备份，3个用于数据分片。

节点最小化配置

```
port 7000
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 5000
appendonly yes
```

启动6个节点，对其中3个节点进行分槽，剩下3个节点分别replicate对应的master节点。

从Redis 5后，可以使用redis-cli --cluster运行快速创建集群的指令。

此外，还有增删节点、重新分片等等运维操作。

需要一个好用的Redis客户端才能使用到cluster的功能，流行语言基本都有比较活跃的client开源。

#### Live Reconfiguration

##### ADDSLOTS, DELSLOTS and SETSLOT

> After the hash slots are assigned they will propagate across the cluster using the gossip protocol, as specified later in the configuration propagation section.
>
> The ADDSLOTS command is usually used when a new cluster is created from scratch to assign each master node a subset of all the 16384 hash slots available.
>
> The DELSLOTS is mainly used for manual modification of a cluster configuration or for debugging tasks: in practice it is rarely used.
>
> The SETSLOT subcommand is used to assign a slot to a specific node ID if the SETSLOT <slot> NODE form is used. Otherwise the slot can be set in the two special states MIGRATING and IMPORTING. Those two special states are used in order to migrate a hash slot from one node to another.
>
> - When a slot is set as MIGRATING, the node will accept all queries that are about this hash slot, but only if the key in question exists, otherwise the query is forwarded using a -ASK redirection to the node that is target of the migration.
> - When a slot is set as IMPORTING, the node will accept all queries that are about this hash slot, but only if the request is preceded by an ASKING command. If the ASKING command was not given by the client, the query is redirected to the real hash slot owner via a -MOVED redirection error, as would happen normally.

#### ASK and MOVED

> Why can't we simply use MOVED redirection? Because while MOVED means that we think the hash slot is permanently served by a different node and the next queries should be tried against the specified node, ASK means to send only the next query to the specified node.
>
> This is needed because the next query about hash slot X can be about a key that is still in A, so we always want the client to try A and then B if needed. Since this happens only for one hash slot out of 16384 available, the performance hit on the cluster is acceptable.
>
> We need to force that client behavior, so to make sure that clients will only try node B after A was tried, node B will only accept queries of a slot that is set as IMPORTING if the client sends the ASKING command before sending the query.
>
> Basically the ASKING command sets a one-time flag on the client that forces a node to serve a query about an IMPORTING slot.
>
> The full semantics of ASK redirection from the point of view of the client is as follows:
>
> - If ASK redirection is received, send only the query that was redirected to the specified node but continue sending subsequent queries to the old node.
> - Start the redirected query with the ASKING command.
> - Don't yet update local client tables to map hash slot X to B.
>
> Clients usually need to fetch a complete list of slots and mapped node addresses in two different situations:
>
> - At startup in order to populate the initial slots configuration.
> - When a MOVED redirection is received.

## Miscellaneous

### Access Control Lists

Starting with version 6 Redis supports ACLs. It is possible to configure users able to run only selected commands and able to access only specific key patterns.

### Lua Scripting and Pipelining

> Using Redis scripting (available in Redis version 2.6 or greater) a number of use cases for pipelining can be addressed more efficiently using scripts that perform a lot of the work needed at the server side. A big advantage of scripting is that it is able to both read and write data with minimal latency, making operations like read, compute, write very fast (pipelining can't help in this scenario since the client needs the reply of the read command before it can call the write command).
>
> Sometimes the application may also want to send EVAL or EVALSHA commands in a pipeline. This is entirely possible and Redis explicitly supports it with the SCRIPT LOAD command (it guarantees that EVALSHA can be called without the risk of failing).

### Lock with a single instance

要考虑一下可重入性，解锁时会不会误解掉别人的锁。注意设置过期时间，耗时任务注意刷新时间，并使用pipeline或者lua脚本来执行解锁操作。

### RedLock

该方案通过客户端来实现逻辑，来同步多个完全独立的实例，不通过replication来实现分布式锁，可以避免single point of failure。这是官方推荐的一种分布式锁方案。