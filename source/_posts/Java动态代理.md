---
title: Java动态代理
date: 2019-09-23 13:35:07
tags: [tech,java]
categories: interview
---

# 背景

学习Spring绕不开的AOP，AOP可以使代码简洁好维护，基本原理就是为对象作代理。我们都知道对于Spring来说可以采用CGLIB和JDK的Proxy实现动态代理。在Spring Boot 2.X之后，已经默认都使用CGLIB来进行代理了。

<!-- more -->

# 总览

### JDK动态代理

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

### CGLIB

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

注意：如果Clazz是接口或抽象类的话对应的抽象方法invokeSuper会失败。

# 总结

JDK动态代理与CGLIB字节码增强在Java8上已经没有质的性能差别，但是CGLIB不需要给被增强类限制为必须实现接口，相对灵活。另一方面，使用JDK代理不需要引入额外的包。

在Spring Boot 2.X中需要配置spring.aop.proxy-target-class=false，否则Spring框架会默认使用CGLIB。

在Spring Boot中尽量不要给方法加final修饰，否则代理类的方法会没有如预期那样得到增强。