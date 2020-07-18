---
title: Miscellaneous
date: 2018-11-20 07:10:28
updated: 2020-07-17 12:00:00
tags: [Miscellaneous]
---

Review questions

<!-- more -->

# Questions

### Heap

```java
void heapify(int[] q) {
    for (int i = q.length >>> 1; i >= 0; i--) {
        siftDown(q, i, q[i]);
    }
}

void siftDown(int[] q, int index, int e) {
    int half = q.length >>> 1;
    while (index < half) {
        int left = (index << 1) + 1;
        int right = q.length == left + 1 ? -1 : left + 1;
        if (right != -1 && q[left] > q[right]) {
            left = right;
        }
        if (e < q[left]) {
            break;
        }
        q[index] = q[left];
        index = left;
    }
    q[index] = e;
}

void siftUp(int[] q, int index, int e) {
    while (index > 0) {
        int parent = (index - 1) >>> 1;
        if (q[parent] < e) {
            break;
        }
        q[index] = q[parent];
        index = parent;
    }
    q[index] = e;
}
```

### 布隆过滤器

利用**布隆过滤器**代替哈希表，利用多Hash函数映射Bit Array的思想，省内存，可以准确判断某项数据不存在。缺点是有一定概率的假正率（误报存在），需要合理定义哈希函数个数，以及比特数组的大小。

### 快排

```java
void quickSort(int[] s, int l, int r){
    if (l < r){
        int i = l, j = r, x = s[l];
        while (i < j){
            while(i < j && s[j] >= x) // 从右向左找第一个小于x的数
                j--;  
            if(i < j) 
                s[i++] = s[j];
			
            while(i < j && s[i] < x) // 从左向右找第一个大于等于x的数
                i++;  
            if(i < j) 
                s[j--] = s[i];
        }
        s[i] = x;
        quickSort(s, l, i - 1); // 递归调用 
        quickSort(s, i + 1, r);
    }
}

```

### LinkedHashMap实现LRU

```java
public class LRUMap<K, V> extends LinkedHashMap<K, V> {
    private int maxSize;

    LRUMap(int maxSize) {
        super(maxSize, 0.75f, true);
        this.maxSize = maxSize;
    }

    @Override
    protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
        return size() > maxSize;
    }
}
```

### 负载均衡算法

轮询、随机、最小连接、源地址哈希、一致性哈希。考虑到服务器处理能力的不同，可以采用加权的策略。

### 一致性哈希

用来解决分布式缓存的Hot Spot问题。将对象与服务器映射到哈希环上，离对象顺时针最近的就是存储的服务器，增删节点只会影响一小段哈希环上的对象。还要注意一下为了解决环偏斜可以引入虚拟节点。

### 数据库分库分表

垂直拆分：根据业务将表按字段拆分。

水平拆分：数据行拆分。

### 分布式锁

数据库乐观锁实现方式：加时间戳或版本号。

Redis实现分布式锁，要考虑一下可重入性，解锁时会不会误解掉别人的锁。




