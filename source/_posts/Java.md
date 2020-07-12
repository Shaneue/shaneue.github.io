---
title: Java
date: 2018-12-03 10:31:40
updated: 2019-07-12 12:00:00
tags: [Language]
typora-root-url: ../
---

Java runtime and features

<!-- more -->

## JVM

### JMM

主内存、工作内存、虚拟机规范、线程模型

### Concurrency

#### 锁优化

##### 自旋锁

自旋等待要占用CPU时间，但是可以避免线程切换的开销，在锁被占用时间很短的情况下效果很好。自旋锁有自旋次数参数，超过这个次数就会挂起线程。此外，还有自适应自旋锁，可以根据上一次锁的自旋次数以及锁的当前拥有者状态来动态调整自旋次数。

##### 锁消除

利用逃逸分析判断数据不会逃逸，被其他线程访问到。这时可以判定数据是线程私有，不需要锁同步。

##### 锁粗化

大部分情形下，总是推荐同步代码块尽量小。但是也有例外的情形，比如一系列操作连续对同一个对象反复加解锁，这时可以使用锁粗化来避免不必要的互斥同步操作。

##### 轻量级锁

在没有竞争的情况下，用CAS操作去修改对象头的Mark Word，标记轻量状态。如果有竞争情况，则膨胀为重量锁。

##### 偏向锁

在锁第一次被线程获取时，会先设置成偏向，如果接下来没有被其他线程获取，则持有偏向锁的线程永远不需要进行同步。当有其他线程尝试获取锁时，偏向模式就结束了。

## Spring

### Spring Boot

## Miscellaneous

### 1. MVC

- 控制器（Controller）- 负责转发请求，对请求进行处理。

- 视图（View） - 界面设计人员进行图形界面设计。

- 模型（Model） - 程序员编写程序应有的功能（实现算法等等）、数据库专家进行数据管理和数据库设计(可以实现具体的功能)。


![](/images/java1.svg)

### 2. Servlet

Servlet是Java Web开发中一个重要的概念，在3.1版本后可以支持非阻塞I/O。

```java
public interface Servlet {
    void init(ServletConfig var1) throws ServletException;

    ServletConfig getServletConfig();

    void service(ServletRequest var1, ServletResponse var2) throws ServletException, IOException;

    String getServletInfo();

    void destroy();
}
```

可以看出最基本的功能就是接收Request与返回Response，并在中间做业务处理。

可以根据这个约定开发实现Servlet。

但是仅仅有Servlet不能支撑起一个服务器，还需要做端口监听，请求封装与返回等操作，一些Servlet容器（可理解为服务器），如Tomcat、Jetty就应运而生了。容器会调用Servlet的接口方法来管理它的生命周期。

Spring Boot中有两个比较重要的上下文类型，一个是ServletWebServer，一个是ReactiveWebServer。在SpringApplication.run的时候会根据classpath进行推测，通常我们做的应用就是ServletWebServer。

### 3. 动态代理

学习Spring绕不开的AOP，AOP可以使代码简洁好维护，基本原理就是为对象作代理。在Spring Boot 2.X之后，已经默认都使用CGLIB来进行代理了。

####  JDK动态代理

JDK反射包中的动态代理主要关注两个类：InvocationHandler和Proxy。

代理逻辑实现在InvocationHandler的invoke方法中。

代理后生成的类是Proxy的子类，并实现了Proxy.newProxyInstance()方法传入的接口。

根据Java单继承的特性，可以猜到，使用JDK的动态代理的类必须实现接口。

```java
public class Handler implements InvocationHandler {
  Original object = new Original();
  @Override
  public Object invoke(Object o, Method method, Object[] args) {
    //第一个参数是代理对象
    method.invoke(object, args);
    return null;
  }
}
```

生成代理对象

```java
Interface object = (Interface) Proxy.newProxyInstance(Handler.class.getClassLoader(), new Class[]{Interface.class}, new Handler());
object.method();
```

#### CGLIB

常用的MethodInterceptor与Enhancer类，MethodInterceptor是Callback的子接口。

MethodInterceptor与InvocationHandler逻辑类似，但是其中的intercept方法比invoke方法多了一个MethodProxy参数，该参数含有一个invokeSuper的方法，可以通过生成子类对象的方式来完成代理。

因为final不可继承，因此对final修饰的方法无法增强。

但是CGLIB其实也可以像JDK代理那样通过使MethodInterceptor持有被代理对象的引用，达到代理final方法的目的（也需要实现接口）。

CGLIB还可以对没有接口的类做代理，因此其实CGLIB的功能比JDK代理要灵活。

```java
public class Interceptor implements MethodInterceptor {
  @Override
  public Object intercept(Object o, Method method, Object[] objects, MethodProxy proxy) throws Throwable {
    //第一个参数是代理对象
    proxy.invokeSuper(o, Objects);
    return null;
  }
}
```

生成代理对象

```java
Enhancer enhancer = new Enhancer;
enhancer.setSuperclass(Clazz.class);
enhancer.setCallback(new Interceptor());
Clazz clazz = (Clazz) enhancer.create();
```

如果Clazz是接口或抽象类的话对应的抽象方法invokeSuper会失败。

在Spring Boot 2.X中需要配置spring.aop.proxy-target-class=false，否则Spring框架会默认使用CGLIB。

### 4. 永久代（PermGen）与元空间（Metaspace）

永久代是HotSpot上的概念，可以理解为方法区的实现。在Java8中已经取消了永久代，改为元空间。永久代是堆内存的一部分，元空间存储类的元数据并使用本地内存，并将永久代中的字符串常量池与类静态变量等放到堆中。

**原因** 一方面是可以减少OOM，最主要的原因是与JRockit虚拟机没有永久代概念，为了融合HotSpot与JRockit。

### 5. Java8 ConcurrentHashMap.size()

几个关键方法与变量：sunCount()、baseCount、CounterCell[] counterCells、addCount()、fullAddCount()

### 6. Fail-fast与Fail-safe

快速失败：在使用迭代器的时候，除了迭代器之外，其它线程对容器的结构做出改变的话，会立即抛出ConcurrentModificationException。

Fail-safe：在使用concurrent包下的容器时，如果并发修改集合结构的话，会首先复制一份数据副本，并在副本上作修改，这样就不会相互影响，但是无法保证读取到的是最新数据。

### 7. AQS原理

CLH队列，用于实现自旋锁

volatile state以及FIFO线程等待队列，state重入次数，0表示未占用

两种资源共享方式：独占（ReentrantLock）、共享（Semaphore、CountDownLatch）

可实现公平与非公平锁

### 8. Arrays.sort() & Collections.sort()

Collections.sort最终会用到Arrays的排序方法。

Arrays的排序方法：当排序元素量小于286、大于47时，使用双轴快排（DualPivotQuickSort），在小数组上（小于47），会使用插入排序。当排序元素量大等于286且有序性好的时候用归并排序（其中归并排序还有传统的归并排序和TimSort，传统归并上面还有注释说“以后会移除”），有序性不好时还是用双轴快排。

### 9. GC ROOTS

虚拟机栈中引用的对象、方法区中类静态属性引用的对象、方法区中常量引用的对象、本地方法栈中引用的对象