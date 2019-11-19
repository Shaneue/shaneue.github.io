---
title: Kubernetes与List-Watch机制
date: 2019-07-03 17:16:53
tags: [tech]
categories: k8s
typora-root-url: ../
---

# 背景

K8s一个重要的组件Etcd是一个Key-value类型的分布式数据库，保存网络、各种资源等的状态信息，也是K8s架构中唯一有状态的组件。其他各种组件的运行需要用到保存在Etcd中的数据，同时，K8s被设计成对Etcd的操作需要经过api-server这个组件来进行。API Server提供了List-Watch的机制来进行高效的异步消息传递，而不需要使用中间件，这种机制是理解K8s架构的基础之一。

<!-- more -->

# 简介

![](/images/k8s_arch.png)

# 原理

### API Server如何提供List&Watch机制

### 如何使用List&Watch自定义Controller

![](/images/list-watch1.png)

![](/images/list-watch2.jpeg)