---
title: Distributed Transaction
date: 2020-07-27 16:10:55
updated: 2020-07-27 17:10:55
tags: [Distributed System]
typora-root-url: ../
---

A database transaction in which two or more network hosts are involved

<!-- more -->

## Two-phase Commit Protocol

In transaction processing, databases, and computer networking, the two-phase commit protocol (2PC) is a type of atomic commitment protocol (ACP). It is a distributed algorithm that coordinates all the processes that participate in a distributed atomic transaction on whether to commit or abort (roll back) the transaction (it is a specialized type of consensus protocol). The protocol achieves its goal even in many cases of temporary system failure (involving either process, network node, communication, etc. failures), and is thus widely used.

### Assumptions

- One node is a designated coordinator, which is the master site, and the rest of the nodes in the network are designated the participants.
- There is stable storage at each node with a write-ahead log, that no node crashes forever, that the data in the write-ahead log is never lost or corrupted in a crash.
- Any two nodes can communicate with each other. 

### Algorithm

#### Commit request phase

- The coordinator sends a query to commit message to all participants and waits until it has received a reply from all participants.
- The participants execute the transaction up to the point where they will be asked to commit. They each write an entry to their undo log and an entry to their redo log.
- Each participant replies with an agreement message (participant votes Yes to commit), if the participant's actions succeeded, or an abort message (participant votes No, not to commit), if the participant experiences a failure that will make it impossible to commit.

#### Commit (or completion) phase

##### Success

If the coordinator received an agreement message from all participants during the commit-request phase:

- The coordinator sends a commit message to all the participants.
- Each participant completes the operation, and releases all the locks and resources held during the transaction.
- Each participant sends an acknowledgement to the coordinator.
- The coordinator completes the transaction when all acknowledgments have been received.

##### Failure

If any participant votes No during the commit-request phase (or the coordinator's timeout expires):

- The coordinator sends a rollback message to all the participants.
- Each participant undoes the transaction using the undo log, and releases the resources and locks held during the transaction.
- Each participant sends an acknowledgement to the coordinator.
- The coordinator undoes the transaction when all acknowledgements have been received.

### Disadvantages

The greatest disadvantage of the two-phase commit protocol is that it is a blocking protocol. If the coordinator fails permanently, some participants will never resolve their transactions: After a participant has sent an agreement message to the coordinator, it will block until a commit or rollback is received.

### Refinements

Three-phase commit protocol, Try-Confirm/Cancel (TCC)

## MQ-based Eventual Consistency

使用数据库和消息队列来保证最终一致性。默认消息队列提供至少一次语义。

步骤：

- 在第一个进程中开始分布式事务时，需要原子性地在数据库中同时记录一条事务记录，事务id唯一。
- 然后扫描数据库表（可以使用定时程序），将下一步的操作封装成消息，并带上事务id发送到消息队列。收到消息发送成功的ack后，将该记录标记为已完成。其中最后一步有可能失败，会导致消息重复发送。
- 接着第二个进程收到消息，执行它在该事务中的任务。执行任务时也需要在数据库表中原子性地插入一条事务，以收到的事务id做主键。这样，每次任务开始时，判断id是否已执行过，提供幂等性。