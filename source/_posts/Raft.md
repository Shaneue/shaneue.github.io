---
title: Raft
date: 2018-11-19 17:43:16
updated: 2019-01-01 12:00:00
tags: [Distributed System]
typora-root-url: ../
---

A consensus algorithm that is designed to be easy to understand

<!-- more -->

### stale read

linearizable read

linearizable write

ReadIndex

CAP

> - Consistency: Every read receives the most recent write or an error
> - Availability: Every request receives a (non-error) response, without the guarantee that it contains the most recent write
> - Partition tolerance: The system continues to operate despite an arbitrary number of messages being dropped (or delayed) by the network between nodes

写请求必须经过leader