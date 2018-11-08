---
title: WEB安全
date: 2018-12-06 20:05:56
tags: [tech]
categories: 实践技术
---

# HTTP

Web是建立在HTTP（HyperText Transfer Protocal，超文本转移协议）协议上通信的。

<!-- more -->

### TCP/IP分层

协议族可以分成应用层（FTP、DNS、**HTTP**）、传输层（TCP、UDP）、网络层（IP）、链路层。

其中跟HTTP关系密切的三个协议：IP、TCP与DNS。

IP（Internet Protocal）：两个重要地址IP地址与MAC地址，MAC用于下一站中转，具有本地意义；而IP地址则主要用于路由。两者通过ARP协议关联起来。

TCP（Transmmision Control Protocal）：为大块数据的传输提供可靠性；将数据切分成报文段（segment）；主要保证通信手段：三次握手（SYN、SYN/ACK、ACK）。

DNS（Domain Name System）：提供域名到IP地址的解析。

### 请求报文

由请求方法（如GET）、请求URI、协议版本、头部（可选）与内容（如Body、参数对）构成。

### 响应

以HTTP版本起头，紧跟状态码（如200 OK）。

接着是响应首部字段（如Date、Content-Length）。

然后响应主体。

### 无状态（stateless）

为了设计简单，处理快速，以及协议的可伸缩性。可以通过Cookie来管理用户状态。

### 持久连接

也称为keep-alive机制。

减少TCP重复建立的开销，如多次请求一个服务器的图片。

HTTP/1.1里，默认使用持久连接。

### pipelining

持久连接可以使多数连接并行，如同时请求10张图片比一张一张请求快。

