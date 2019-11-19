---
title: 分布式全局ID
date: 2018-11-19 17:43:16
tags: [tech]
categories: 实践技术
typora-root-url: ../
---

# 简介

### 分布式全局ID需要具备的特点

最常见的需求是全局唯一，并且还具有递增的特性，这也是[Paxos算法](https://zh.wikipedia.org/wiki/Paxos算法)中proposal number的基本需求。此外，可能还要保证ID生成功能的高可用性，最好还要带有时间戳来表明ID的生成时间，而且长度适合，不要增加存储压力，并且适合拿来做索引。

<!-- more -->

### 常见需求场景

1. 如前面提到的一致性算法Paxos，在prepare阶段就需要一个全局唯一并且递增的proposal number；
2. 实现分布式环境下接口的幂等性；
3. 记录分布式请求的调用链；
4. 实现数据库多版本并发控制（[MVCC](https://en.wikipedia.org/wiki/Multiversion_concurrency_control)，Multi-Version Concurrency Control），每个数据行都有一个递增的全局唯一ID；
5. 数据库分库分表需要全局主键；

# 常见解决方案

### Snowflake

![](/images/snowflake.jpg)

根据Twitter的业务需求，Snowflake系统生成64位的ID。由3部分组成：

- 41位的时间序列（精确到毫秒，41位的长度可以使用69年）
- 10位的机器标识（10位的长度最多支持部署1024个节点）
- 12位的计数顺序号（12位的计数顺序号支持每个节点每毫秒产生4096个ID序号）

主要缺点：依赖机器时钟，有时钟回拨的风险。现在这个系统已经不维护了。

### 美团Leaf

Leaf采用了和Snowflake类似的ID结构，对Snowflake方案做了改进，并考虑了时钟回拨的问题。但是Leaf要依赖数据库与ZooKeeper。
