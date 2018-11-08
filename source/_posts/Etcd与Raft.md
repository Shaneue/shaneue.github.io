---
title: Etcd与Raft
date: 2019-07-03 17:17:35
tags: [tech]
categories: k8s
---

# 背景

想要了解Etcd主要是因为它是K8s分布式特性的主要提供者。Etcd也是一个满足强一致性的组件，通过健值对的方式对外提供数据存储与访问。Etcd实现了Raft一致性算法。

<!-- more -->

# Raft

#### 节点的三种状态

Follower、Candidate、Leader

对系统的更新操作需要经过Leader节点，当所有节点达到一致性后，可请求任意节点获取数据。

#### 选举概述

- Follower如果接收不到Leader的心跳会变成Candidate
- Candidate向其他节点请求投票
- 如果获取的票数过半则成为Leader
- 两个超时时间，一个叫election timeout，另一个是heartbeat timeout
- 选举超时，则投票给自己，term+1，并向其他节点请求投票（Request Vote），并重置选举超时时间
- 成为Leader后，开始不间断地发送Append Entries，根据心跳超时来控制
- 如果没接收到心跳，则Follower会变成Candidate，并开始重新选举

#### 更新数据（Log Replication）

首先向Leader发送命令，Leader写日志entry，但是未提交。然后向其他节点发送这个日志entry，等待大多数节点已经写下这个日志，该Leader即可提交更改并且通知其他节点。

更新操作使用的也是Append Entries，随着心跳机制发送。

#### 网络分区

网络分区之后只有其中一个分区有能力提交变更，因为仅有一个分区的节点数量过半。

分区解除后，无法提交变更的那个分区会去同步已变更的日志。