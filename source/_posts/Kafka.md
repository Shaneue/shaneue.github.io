---
title: Kafka
date: 2020-06-06 13:50:03
updated: 2020-08-01 12:00:00
tags: [Message Broker]
typora-root-url: ../
---

A distributed streaming platform

<!-- more -->

## CONFIG

- zookeeper.connect=localhost:2181（Kafka依赖于zookeeper，默认请求本地2181端口的zookeeper服务）
- broker.id=0（在cluster中标识自己，默认为0，在启动多个Kafka服务时需要修改）
- listeners=PLAINTEXT://:9092（端口号默认9092，在本机启动多个Kafka服务时需要修改）
- log.dirs=/tmp/kafka-logs（在本机启动多个Kafka服务时需要修改）

### consumer.properties

- enable.auto.commit=true（消费者取到消息后先commit再进行消费）
- isolation.level

### producer.properties

- acks=1（leader会立即返回ack，-1或者all表示leader会等待in-sync replicas确认后再返回ack，0表示不需要ack）
- transactional.id
- enable.idempotence
- max.in.flight.requests.per.connection（设置成1，可以避免乱序，同时牺牲了效率，默认为5）

## USAGE SCENARIO

![](/images/kafka1.png)


- 在系统与应用之间构建实时数据传输
- 实时对数据流进行处理加工，重新发布到topic
- 监听某些系统的数据变化

## CONCEPTS

一个topic会拆分为多个partition

![](/images/kafka2.png)

broker、partition、consumer、group的关系

![](/images/kafka3.png)

## DESIGN

### 1. Filesystem

恰当使用文件系统可以使磁盘I/O效率不低于网络I/O。

源码中使用了FileChannel.write(ByteBuffer)来为segment文件追加Record。

> Linear writes on a JBOD configuration with six 7200rpm SATA RAID-5 array is about 600MB/sec but the performance of random writes is only about 100k/sec.

现代操作系统会使用所有空闲内存来做磁盘cache，因为回收内存的开销很小。这样就可以使用read-ahead与write-behind技术来预读，以及批量写入。

不使用Java对象来做内存cache，一方面Java对象的内存开销是是数据内容的两倍或更糟，另一方面GC会随着堆数据的增加变得很不友好。

Disk seek很慢且无法并行。

围绕pagecache做设计。

使用持久化队列的数据结构存储，所有操作都是常数复杂度，无删除操作，读与写不会相互阻塞。

### 2. Efficiency

磁盘效率得到保证后，大量小的I/O操作以及过多的字节复制会影响效率。

I/0操作包括client与server之间的请求以及server的持久化，解决方法就是使用Batching，分摊网络开销与磁盘操作的开销。

解决字节复制，首先需要所有组件共享同一种二进制消息格式，数据转发、持久化不需要做修改。在标准格式的前提下，不需要使用read，可以使用sendfile系统调用来减少复制。

#### 通常情况的数据拷贝路径：

1. The operating system reads data from the disk into pagecache in kernel space
2. The application reads the data from kernel space into a user-space buffer
3. The application writes the data back into kernel space into a socket buffer
4. The operating system copies the data from the socket buffer to the NIC buffer where it is sent over the network

#### sendfile

该系统调用是Linux特有。

利用DMA技术，只需要从pagecache复制到NIC buffer。

Java的零拷贝技术可以使用java.nio.channels.FileChannel的transferTo()。

#### mmap

该系统调用是POSIX兼容的。

在Java中对应java.nio.MappedByteBuffer，由FileChannel的map方法产生。

#### Batch Compression

消息批量压缩的效率更高，且只会被consumer解压。

### 3. Producer

- Producer直接往partition的leader写数据。如果请求的是node，node会告诉你leader的地址。
- Producer可以指定partition写数据，比如将所有的数据写到同一个partition。
- Producer可配置压缩或者batch size来发送数据，以加快效率。
- Producer采用Push模式。

### 4. Consumer

- Consumer采用Pull模式。
- 追踪消费进度，一个partition的一个consumer只保存一个offset即可。
- Offset数据存储在*__consumer_offsets*的topic中。

## REPLICATION

默认配置下，是一个满足AP的系统。

### Zookeeper

2.5版本对Zookeeper的依赖变少了。

#### 1. Broker Node Registry

```
/brokers/ids/[0...N] --> {``"jmx_port"``:...,``"timestamp"``:...,``"endpoints"``:[...],``"host"``:...,``"version"``:...,``"port"``:...} (ephemeral node)
```

> Since the broker registers itself in ZooKeeper using ephemeral znodes, this registration is dynamic and will disappear if the broker is shutdown or dies (thus notifying consumers it is no longer available).

#### 2. Broker Topic Registry

```
`/brokers/topics/[topic]/partitions/[0...N]/state --> {``"controller_epoch"``:...,``"leader"``:...,``"version"``:...,``"leader_epoch"``:...,``"isr"``:[...]} (ephemeral node)`
```

> Each broker registers itself under the topics it maintains and stores the number of partitions for that topic.

#### 3. Cluster Id

/cluster/id是一个自动生成的id唯一标识cluster

### In-Sync Replica

This ISR set is persisted to ZooKeeper whenever it changes. Because of this, any replica in the ISR is eligible to be elected leader.

1. A node must be able to maintain its session with ZooKeeper (via ZooKeeper's heartbeat mechanism)
2. If it is a follower it must replicate the writes happening on the leader and not fall "too far" behind

We refer to nodes satisfying these two conditions as being "in sync".

Kafka使用ISR来提供主从partition的数据一致性与failover。

## MISCELLANEOUS

### 1. At-least-once

默认acks=1，可以实现至少一次消息提交。似乎还是会因为宕机且没有fsync丢失数据。

### 2. Exactly-once

提供了幂等性与事务配置来实现Producer的Exactly-once语义。

对消费者来说，需要关闭offset的自动提交，自己来保证只消费一次。比如消费完再提交offset，并且有办法辨别重复消费。

### 3. Rebalance

> Kafka provides the guarantee that a partition in a topic is assigned to only one consumer within a group.
>
> The ability of consumers clients to cooperate within a dynamic group is made possible by the use of the so-called Rebalance Protocol.

0.9.0版本前，需要依赖Zookeeper来记录consumer的信息，造成zk的频繁写。

现版本中，这部分信息会存在__consumer_offsets上，并使用broker上的group coordinator来进行rebalance。

### 4. Static Membership

利用这个特性，可以减少触发rebalance过程的次数。

> If you want to use static membership,
>
> - Upgrade both broker cluster and client apps to 2.3 or beyond, and also make sure the upgraded brokers are using inter.broker.protocol.version of 2.3 or beyond as well.
> - Set the config ConsumerConfig GROUP_INSTANCE_ID_CONFIG to a unique value for each consumer instance under one group.
> - For Kafka Streams applications, it is sufficient to set a unique ConsumerConfig GROUP_INSTANCE_ID_CONFIG per KafkaStreams instance, independent of the number of used threads for an instance.

### 5. Group Coordinator

> Kafka provides the option to store all the offsets for a given consumer group in a designated broker (for that group) called the group coordinator.