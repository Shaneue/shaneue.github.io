---
title: Stereotyped Writings of Interview
date: 2017-07-07 18:00:00
updated: 2017-07-08 18:00:00
tags: [Interview]
typora-root-url: ../
---

Eight-legged Essay

<!-- more -->

### 红黑树 VS AVL

AVL适合查找密集型、红黑树适合插入修改密集型、红黑树最多只需要三次旋转即可达到平衡，avl是严格的平衡二叉树，在现实生产中红黑树的统计出来的性能更好，稳定性好。

### AQS

aqs是java并发包里基础的一个类，很多并发类基于该抽象类来实现。该类主要提供了独占/共享、公平/非公平锁实现的拓展。

该类基于CLH队列，是CLH队列的一种变种实现。CLH主要用于自旋锁的实现，让线程在上面等待资源释放。

CLH是一个FIFO的双向链表。

实现aqs时，公平与非公平主要在于tryAcquire时，会去判断队列是否有等待线程在，如果有的话acquire失败会选择排到队尾。

state在共享模式时，表示共享次数，独占模式时，表示重入次数。

读锁共享，交替重入的情况，使缓存失效。

### JVM内存空间

线程私有：程序计数器、虚拟机栈、本地方法栈

堆：元空间、堆

堆：Eden、Survivor(from, to)、Old

### TLAB（Thread Local Allocation Buffer）

CAS

### 缓存数据库一致

最简单的方法可以采用先更新数据库再删除缓存，在改完库到更新缓存前，请求会得到旧数据。

延时双删可以将数据不一致性的时间窗口降低。

利用中间件通过mq监听数据库更新，在业务架构上可以解耦。

### Redis单实例锁

可重入、设置过期时间、刷新过期时间、lua脚本解锁

分布式锁可以使用官方推荐的RedLock

### Spring循环依赖

递归？三级缓存，构造器上的循环依赖直接报错

### Spring Boot启动过程

记不住= =

大概就是扫描各个包的spring.factories，计时器、打印Bannner、准备环境，准备各种RunListener，基础组件像日志啥的，还要准备beanFactory啥的，有个很重要的方法叫refresh()要执行一下，各种Listener跑起来并监听它们，成功跑完之后会打印一个启动时间。

### HashMap.put

```java
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    else {
        Node<K,V> e; K k;
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        else if (p instanceof TreeNode)
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else {
            for (int binCount = 0; ; ++binCount) {
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    break;
                }
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        if (e != null) { // existing mapping for key
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
    ++modCount;
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    return null;
}
```

### ConcurrentHashMap的线程安全

synchronized+cas，会锁住Node f，不再使用Segment，具体过程比较复杂，记不住，可能得去看看源码

### 乐观锁、悲观锁

Lock、CAS，ABA问题，加版本号或时间戳。

### 父子线程共享数据

InheritableThreadLocal

### AOP

切面的工作被称为通知。如果通知定义了“什么”和“何时”，那么切点就定义了“何处”。

切面是通知和切点的集合，通知和切点共同定义了切面的全部功能——它是什么，在何时何处完成其功能。

### 数组原地打乱

Collections.shuffle

### SQL指定索引

### 线程池

```java
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          ThreadFactory threadFactory,
                          RejectedExecutionHandler handler) {
    if (corePoolSize < 0 ||
        maximumPoolSize <= 0 ||
        maximumPoolSize < corePoolSize ||
        keepAliveTime < 0)
        throw new IllegalArgumentException();
    if (workQueue == null || threadFactory == null || handler == null)
        throw new NullPointerException();
    this.acc = System.getSecurityManager() == null ?
            null :
            AccessController.getContext();
    this.corePoolSize = corePoolSize;
    this.maximumPoolSize = maximumPoolSize;
    this.workQueue = workQueue;
    this.keepAliveTime = unit.toNanos(keepAliveTime);
    this.threadFactory = threadFactory;
    this.handler = handler;
}
```

```java
public void execute(Runnable command) {
    if (command == null)
        throw new NullPointerException();
    /*
     * Proceed in 3 steps:
     *
     * 1. If fewer than corePoolSize threads are running, try to
     * start a new thread with the given command as its first
     * task.  The call to addWorker atomically checks runState and
     * workerCount, and so prevents false alarms that would add
     * threads when it shouldn't, by returning false.
     *
     * 2. If a task can be successfully queued, then we still need
     * to double-check whether we should have added a thread
     * (because existing ones died since last checking) or that
     * the pool shut down since entry into this method. So we
     * recheck state and if necessary roll back the enqueuing if
     * stopped, or start a new thread if there are none.
     *
     * 3. If we cannot queue task, then we try to add a new
     * thread.  If it fails, we know we are shut down or saturated
     * and so reject the task.
     */
    int c = ctl.get();
    if (workerCountOf(c) < corePoolSize) {
        if (addWorker(command, true))
            return;
        c = ctl.get();
    }
    if (isRunning(c) && workQueue.offer(command)) {
        int recheck = ctl.get();
        if (! isRunning(recheck) && remove(command))
            reject(command);
        else if (workerCountOf(recheck) == 0)
            addWorker(null, false);
    }
    else if (!addWorker(command, false))
        reject(command);
}
```

### 拒绝策略

AbortPolicy, DiscardPolicy, DiscardOldestPolicy, CallerRunsPolicy, self-define

### SQL优化

explain。倒不如准备足够的数据，测试😁

### Java对象创建过程

类检查、分配内存、零值、设置对象头、执行init

### 单例模式

```java
public class Singleton {
    private static Singleton singleton;

    private Singleton() {
    }

    public static Singleton getInstance() {
        if (singleton == null) {
            synchronized (Singleton.class) {
                if (singleton == null) {
                    System.out.println("i m in new instance!");
                    singleton = new Singleton();
                }
            }
        }
        return singleton;
    }
}
```

### 数据库分库分表中间件原理

### mq延迟消息

TimeWheel时间轮算法

### 线程池配置技巧

根据计算密集还是io密集型任务来配置size

### Spring Boot

ComponentScan、EnableAutoConfiguration、Import、DependsOn、Conditional

### 阻塞队列

参考LinkedBlockingQueue的实现

### UUID

V1-V5，其实使用V4就行了，每秒100亿，持续生产100年，产生一次重复的概率是50%。如果想要产生的uuid有序，需要采用时间作为前缀。

### GC调优

新生代大小、大对象大小、进入老年代年龄大小、堆大小、监控minor gc次数与时间（10s一次、50ms）、full gc次数与时间（10分钟一次、1s）

### 限流

服务端、客户端、令牌、漏桶

### Netty线程模型

EventLoop、Selector、Channel、boss、worker、EventLoopGroup

