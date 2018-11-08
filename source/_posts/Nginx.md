---
title: Nginx
date: 2018-12-05 19:08:47
tags: [nginx]
categories: 开源应用
---

# 简介

Nginx是一个轻量级的Web反向代理服务器，主要特点占用内存少，并发能力强。

### 关键词

epoll

<!-- more -->

# 常用操作

	记得先将nginx/bin目录加到path中

启动：nginx

制定配置文件启动：nginx -c *.conf

停止：nginx -s stop

重启：nginx -s reload

用其他配置文件重启：nginx -s reload -c *.conf

# 配置

worker_processes # 可设置成cpu数

error_log

user nginx

pid

worker_rlimit_nofile # 最大文件打开数

worker_cpu_affinity # cpu亲和力

……

event {

​	use epoll;

​	……

}

http {

​	……

​	server {

​		……

​	}

}

# 文件服务器

http {

​	autoindex on;

​	autoindex_exact_size on;

​	autoindex_localtime on;

​	server {

​		listen	80;

​		root	/path

​	}

}

> 需要父目录权限，可以设为755

# 多台服务器代理

传统架构扩容，需要部署多台一样的应用服务器。代理方面可使用upstream来实现，比如客户端ip哈希映射到其中一台服务器。

# 自定义选择服务器

比如需要指定下载某一台服务器的日志。可在头部自定义变量，Nginx可解析头部是否含有对应字段，自定义转发至指定的服务器。