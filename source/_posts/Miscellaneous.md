---
title: Miscellaneous
date: 2018-11-20 07:10:28
updated: 2020-010-17 12:00:00
tags: [Miscellaneous]
typora-root-url: ../
---

Review questions

<!-- more -->

# Questions

### KMP

```java
int[] next(char[] p) {
    int[] next = new int[p.length];
    next[0] = -1;
    int i = 0, j = -1;
    while (i < p.length - 1) {
        if (j == -1 || p[i] == p[j]) {
            i++;
            j++;
            if (p[i] != p[j]) next[i] = j;
            else next[i] = next[j];
        } else j = next[j];
    }
    return next;
}

int indexOf(String source, String pattern) {
    char[] s = source.toCharArray();
    char[] p = pattern.toCharArray();
    if (s.length < p.length) return -1;
    int i = 0, j = 0;
    int[] next = next(p);
    while (i < s.length) {
        if (j == -1 || s[i] == p[j]) {
            i++;
            j++;
        } else j = next[j];
        if (j == p.length) break;
    }
    return j == p.length ? i - j : -1;
}
```

### Greatest Common Divisor

```java
int gcd(int m, int n) {
    while (n != 0) {
        m %= n;
        m ^= n;
        n ^= m;
        m ^= n;
    }
    return m;
}
```

### 逆元

- #### 递推公式：

```java
inv[i] = inv[p % i] * (p - p / i) % p
```

- #### Extended Euclid

- #### 快速幂

```java
long inverseElement(long a, long n) {
    return quickPower(a, n - 2, n);
}

long quickPower(long a, long b, long mod) {
    long ret = 1;
    while (b != 0) {
        if ((b & 1) == 1) ret = ret * a % mod;
        a = a * a % mod;
        b >>= 1;
    }
    return (int) (ret % mod);
}
```

### Merge Sort

```java
void mergeSort(int[] nums, int l, int r, int[] temp) {
    if (l == r) return;
    int mid = l + (r - l) / 2;
    mergeSort(nums, l, mid, temp);
    mergeSort(nums, mid + 1, r, temp);
    int i = l, j = mid + 1, s = l;
    while (i <= mid && j <= r) temp[s++] = nums[i] < nums[j] ? nums[i++] : nums[j++];
    while (i <= mid) temp[s++] = nums[i++];
    while (j <= r) temp[s++] = nums[j++];
    for (i = l; i <= r; i++) nums[i] = temp[i];
}
```

### Disjoint Set

```java
public class DisjointSet {
    int[] f;

    public DisjointSet(int n) {
        this.f = new int[n];
        for (int i = 0; i < n; i++) {
            f[i] = i;
        }
    }

    public void union(int p, int q) {
        int rootP = root(p);
        int rootQ = root(q);

        if (rootP != rootQ) {
            f[rootP] = f[rootQ];
        }
    }

    public boolean find(int p, int q) {
        return root(p) == root(q);
    }

    private int root(int i) {
        if (i != f[i]) {
            // path compression
            f[i] = root(f[i]);
        }
        return f[i];
    }
}
```

### Binary Indexed Tree

```java
public class Bit {
    int[] array;

    public Bit(int length) {
        array = new int[length + 1];
    }

    public void update(int x, int v) {
        for (; x < array.length; x += x & -x) {
            array[x] += v;
        }
    }

    public int query(int x) {
        int sum = 0;
        for (; x > 0; x -= x & -x) {
            sum += array[x];
        }
        return sum;
    }
}
```

### LFU Cache

```java
public class LFUCache {
    static class Node {
        Node prev, next;
        int key, value;
        int frequency;

        Node(int key, int value) {
            this.key = key;
            this.value = value;
        }
    }

    int capacity;
    Node head = new Node(0, 0);
    Node tail = new Node(0, 0);
    Map<Integer, Node> keyMap = new HashMap<>();
    Map<Integer, Node> frequencyMap = new HashMap<>();
    int size = 0;

    public LFUCache(int capacity) {
        this.capacity = capacity;
        head.frequency = 0;
        tail.frequency = Integer.MAX_VALUE;
        head.next = tail;
        tail.prev = head;
        frequencyMap.put(head.frequency, head);
        frequencyMap.put(tail.frequency, tail);
    }

    public int get(int key) {
        Node node = keyMap.get(key);
        if (node == null) {
            return -1;
        }
        promote(node);
        return node.value;
    }

    public void put(int key, int value) {
        if (capacity == 0) return;
        Node node = keyMap.get(key);
        if (node == null) {
            if (size >= capacity) {
                rmLFU();
                size--;
            }
            Node nodeNew = new Node(key, value);
            nodeNew.frequency = 1;
            Node frequencyOne = frequencyMap.get(1);
            if (frequencyOne == null) {
                insertAfter(head, nodeNew);
            } else {
                insertAfter(frequencyOne, nodeNew);
            }
            size++;
            frequencyMap.put(1, nodeNew);
            keyMap.put(key, nodeNew);
        } else {
            promote(node);
            node.value = value;
        }
    }

    void remove(Node node) {
        Node prev = node.prev;
        Node next = node.next;
        prev.next = next;
        next.prev = prev;
    }

    void insertAfter(Node who, Node node) {
        Node next = who.next;
        who.next = node;
        node.next = next;
        next.prev = node;
        node.prev = who;
    }

    void promote(Node node) {
        int f = node.frequency;
        node.frequency++;
        Node prev = node.prev;
        Node list = frequencyMap.get(f);
        Node nextList = list.next;
        if (node == list) {
            if (prev.frequency == f) {
                frequencyMap.put(f, prev);
            } else {
                frequencyMap.remove(f);
            }
            if (nextList.frequency == f + 1) {
                remove(node);
                insertAfter(frequencyMap.get(f + 1), node);
            }
        } else {
            remove(node);
            if (nextList.frequency == f + 1) {
                insertAfter(frequencyMap.get(f + 1), node);
            } else {
                insertAfter(list, node);
            }
        }
        frequencyMap.put(f + 1, node);
    }

    void rmLFU() {
        Node node = head.next;
        Node next = node.next;
        remove(node);
        keyMap.remove(node.key);
        if (node.frequency != next.frequency) {
            frequencyMap.remove(node.frequency);
        }
    }
}
```

### Why use 2s complement

一个是方便正负数相加，另一个是使0只有一种表示。

### TCP

![](/images/tcp.gif)

### Binary Search

```java
int binarySearch(int[] array, int v) {
    int l = 0, r = array.length - 1;
    while (l <= r) {
        int m = (l + r) / 2;
        if (array[m] > v) {
            r = m - 1;
        } else if (array[m] < v) {
            l = m + 1;
        } else {
            return m;
        }
    }
    return -1;
}
```

### Reverse Linked List

```java
Node reverseLinkedList(Node list) {
    Node head = null, t;
    while (list != null) {
        t = list.next;
        list.next = head;
        head = list;
        list = t;
    }
    return head;
}
```

### Heap

```java
void heapify(int[] q) {
    for (int i = (q.length >>> 1) - 1; i >= 0; i--) {
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

### Bloom Filter

利用**布隆过滤器**代替哈希表，利用多Hash函数映射Bit Array的思想，省内存，可以准确判断某项数据不存在。缺点是有一定概率的假正率（误报存在），需要合理定义哈希函数个数，以及比特数组的大小。

注意选择哈希函数个数与过滤器长度。

### Quick Sort

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

### LRU using LinkedHashMap

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

### Load Balancing

轮询、随机、最小连接、源地址哈希、一致性哈希、Hash Slot。考虑到服务器处理能力的不同，可以采用加权的策略。

### Consistent Hash

用来解决分布式缓存的Hot Spot问题。将对象与服务器映射到哈希环上，离对象顺时针最近的就是存储的服务器，增删节点只会影响一小段哈希环上的对象。还要注意一下为了解决环偏斜可以引入虚拟节点，有点类似哈希槽。该算法适合用红黑树实现。
