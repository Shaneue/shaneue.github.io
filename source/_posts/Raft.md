---
title: Raft
date: 2018-11-19 17:43:16
updated: 2020-04-01 12:00:00
tags: [Distributed System]
typora-root-url: ../
---

A consensus algorithm that is designed to be easy to understand

<!-- more -->

## Basics

CAP

> - Consistency: Every read receives the most recent write or an error
> - Availability: Every request receives a (non-error) response, without the guarantee that it contains the most recent write
> - Partition tolerance: The system continues to operate despite an arbitrary number of messages being dropped (or delayed) by the network between nodes

Raft implements consensus by first electing a distinguished leader, then giving the leader complete responsibility for managing the replicated log. The leader accepts log entries from clients, replicates them on other servers, and tells servers when it is safe to apply log entries to their state machines. 

Given the leader approach, Raft decomposes the consensus problem into three relatively independent subproblems:

- Leader election: a new leader must be chosen when an existing leader fails.

- Log replication: the leader must accept log entries from clients and replicate them across the cluster, forcing the other logs to agree with its own.

- Safety: the key safety property for Raft is the State Machine Safety Property. If any server has applied a particular log entry to its state machine, then no other server may apply a different command for the same log index.

The leader handles all client requests (if a client contacts a follower, the follower redirects it to the leader). 

## Guarantees

- Election Safety: at most one leader can be elected in a given term.
- Leader Append-Only: a leader never overwrites or deletes entries in its log; it only appends new entries. 
- Log Matching: if two logs contain an entry with the same index and term, then the logs are identical in all entries up through the given index.
- Leader Completeness: if a log entry is committed in a given term, then that entry will be present in the logs of the leaders for all higher-numbered terms.
- State Machine Safety: if a server has applied a log entry at a given index to its state machine, no other server will ever apply a different log entry for the same index.

## Election

Raft servers communicate using remote procedure calls (RPCs), and the basic consensus algorithm requires only two types of RPCs. RequestVote RPCs are initiated by candidates during elections, and AppendEntries RPCs are initiated by leaders to replicate log entries and to provide a form of heartbeat.

If a follower receives no communication over a period of time called the election timeout, then it assumes there is no viable leader and begins an election to choose a new leader.

Raft uses randomized election timeouts to ensure that split votes are rare and that they are resolved quickly.

## Log Replication

In Raft, the leader handles inconsistencies by forcing the followers’ logs to duplicate its own. This means that conflicting entries in follower logs will be overwritten with entries from the leader’s log.

## Algorithm and Rules

![](/images/raft1.png)

![](/images/raft2.png)

![](/images/raft3.png)

![](/images/raft4.png)

## Membership Changes

Unfortunately, any approach where servers switch directly from the old configuration to the new configuration is unsafe. It is not possible to atomically switch all of the servers at once, so the cluster can potentially split into two independent majorities during the transition.

In order to ensure safety, configuration changes must use a two-phase approach. There are a variety of ways to implement the two phases. For example, some systems use the first phase to disable the old configuration so it cannot process client requests; then the second phase enables the new configuration.

In Raft the cluster first switches to a transitional configuration we call joint consensus; once the joint consensus has been committed, the system then transitions to the new configuration.

论文中描述的方式支持一次变更多个成员，不过工程上不常用，etcd并不采用这种方式。