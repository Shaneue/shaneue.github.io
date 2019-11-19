---
title: SSM
date: 2018-12-03 10:31:40
tags: [ssm]
categories: 框架
typora-root-url: ../
---

# 简介

提供Spring的Bean管理机制、切面编程，Spring MVC使WEB应用接口开发更容易，MyBatis提供关系对象映射。整合起来可以更方便地开发WEB应用。

<!-- more -->

# Spring与Spring MVC

Spring起先只是一个IoC容器，后来因为太好用以及社区越来越强大，慢慢地整合了一些应用开发的框架，并且渐渐在Java届占据了主导地位。

Spring MVC就是一个基于Spring基础的一套快速开发WEB应用接口的框架，基于Servlet的分发（DispatcherServlet）展开MVC架构。

### MVC

- 控制器（Controller）- 负责转发请求，对请求进行处理。

- 视图（View） - 界面设计人员进行图形界面设计。

- 模型（Model） - 程序员编写程序应有的功能（实现算法等等）、数据库专家进行数据管理和数据库设计(可以实现具体的功能)。

  <center>(摘录自维基百科)</center>

![](/images/MVC-Process.svg)

### Spring MVC核心内容

MVC最大的改进是将传统的Model拆分出来一个Service层。在这个层里，可以调用些其他服务并封装提供服务给其他层，比如实现中间件的调用与封装，可以使框架更灵活一点，满足更多元的场景。