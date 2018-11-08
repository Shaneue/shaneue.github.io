---
title: Servlet
date: 2018-12-03 13:43:10
tags: [tech]
categories: Java
---

# 简介

Servlet是Java Web的核心。其实只是一个很简单的规范，或者直接说接口。

```java
public interface Servlet {
    void init(ServletConfig var1) throws ServletException;

    ServletConfig getServletConfig();

    void service(ServletRequest var1, ServletResponse var2) throws ServletException, IOException;

    String getServletInfo();

    void destroy();
}
```

<!-- more -->

可以看出最基本的功能就是接收Request与返回Response，并在中间做业务处理。

可以根据这个约定开发实现Servlet。

但是仅仅有Servlet不能支撑起一个服务器，还需要做端口监听，请求封装与返回等操作，一些Servlet容器（可理解为服务器），如Tomcat、Jetty就应运而生了。容器会调用Servlet的接口方法来管理它的生命周期。

学习Servlet容器的原理才比较重要，不过同样，现在的框架成熟度使开发人员也渐渐不需要接触这些东西了。

# Servlet版本简介