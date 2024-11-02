---
layout:     post

title:      "ConcurrentHashMap源码分析"
# subtitle:   "Java并发编程的艺术笔记"
# excerpt: "Java并发编程的艺术笔记"
author:     "谢文进"
date:       2022-05-06
description: "分析ConcurrentHashMap源码。"
# image: "/img/2022-03-28-The-Art-of-Java-Concurrency-Programming/background.jpg"
published: 2022-05-06 
tags:
     - Java
     - ConcurrentHashMap

categories: [ Tech ]
URL: "/2022/05/06/ConcurrentHashMap/"
---
## JDK 1.7
### 构造方法
```java
public ConcurrentHashMap(int initialCapacity,
                             float loadFactor, int concurrencyLevel) {
    // 检查输入参数的合法性
    if (!(loadFactor > 0) || initialCapacity < 0 || concurrencyLevel <= 0)
        throw new IllegalArgumentException();
    if (concurrencyLevel > MAX_SEGMENTS)
        concurrencyLevel = MAX_SEGMENTS; // MAX_SEGMENTS = 1 << 16, 即 2 ^ 16
    // Find power-of-two sizes best matching arguments
    // sshift 是 ssize 从 1 向左移位的次数
    int sshift = 0;
    // segments 的长度，是 2 的 N 次方
    int ssize = 1;
    // 计算出一个大于等于 concurrencyLevel 的最小的 2 的 N 次方的值作为 segments 的长度
    while (ssize < concurrencyLevel) {
        ++sshift;
        ssize <<= 1;
    }
    // segmentShift 用于定位参与散列运算的位数
    this.segmentShift = 32 - sshift;
    // 散列运算的掩码
    this.segmentMask = ssize - 1;
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;

    /* 
       cap 是 segment 里 HashEntry 数组的长度，
       cap 等于 initialCapacity 除以 sszie 的倍数 c ，
       如果 c 大于 2，就会取大于等于 c 的 2 的 N 次方值，
       所以 cap 不是 2，就是 2 的 N 次方。
    */
    int c = initialCapacity / ssize;
    if (c * ssize < initialCapacity)
        ++c;
    int cap = MIN_SEGMENT_TABLE_CAPACITY; // MIN_SEGMENT_TABLE_CAPACITY = 2
    while (cap < c)
        cap <<= 1;

    // create segments and segments[0]
    Segment<K,V> s0 =
        new Segment<K,V>(loadFactor, (int)(cap * loadFactor),
                            (HashEntry<K,V>[])new HashEntry[cap]);
    Segment<K,V>[] ss = (Segment<K,V>[])new Segment[ssize];
    // 把 s0 存到 Segment 数组中，仅仅创建了一个 Segment 用来作原型对象。
    UNSAFE.putOrderedObject(ss, SBASE, s0); // ordered write of segments[0]
    this.segments = ss;
}
```
### `put()`方法
```java
// Map 的 put 方法
public V put(K key, V value) {
    Segment<K,V> s;
    // value 不能为空
    if (value == null)
        throw new NullPointerException();
    // 计算 hash 值
    int hash = hash(key);
    /*
    j 为 Segment 中的下标
    默认情况下 segmentShift 为 28，segmentMask 为 15
    hash >>> segmentShift 向右无符号移动 28 位，意思是让高 4 位参与到散列运算中
    然后和掩码 segmentMask 做与运算，得到的值一定是 0000 ～ 1111 范围内的值，即 0 ～ 15
    */  
    int j = (hash >>> segmentShift) & segmentMask;
    if ((s = (Segment<K,V>)UNSAFE.getObject          // nonvolatile; recheck
            (segments, (j << SSHIFT) + SBASE)) == null) //  in ensureSegment
        s = ensureSegment(j);
    return s.put(key, hash, value, false);
}
```
补充上述源码中的右移与无符号右移的区别：
* 右移>> ：该数对应的二进制码整体右移，左边的用原有标志位补充，右边超出的部分舍弃。
* 无符号右移>>> ：不管正负标志位为0还是1，将该数的二进制码整体右移，左边部分总是以0填充，右边部分舍弃。

在上述`put()`方法中，hash方法为：
```java
private int hash(Object k) {
    /*
    A randomizing value associated with this instance that is applied to
    hash code of keys to make hash collisions harder to find.

    private transient final int hashSeed = randomHashSeed(this);
    */
    int h = hashSeed;

    if ((0 != h) && (k instanceof String)) {
        return sun.misc.Hashing.stringHash32((String) k);
    }

    h ^= k.hashCode();

    // Spread bits to regularize both segment and index locations,
    // using variant of single-word Wang/Jenkins hash.
    h += (h <<  15) ^ 0xffffcd7d;
    h ^= (h >>> 10);
    h += (h <<   3);
    h ^= (h >>>  6);
    h += (h <<   2) + (h << 14);
    return h ^ (h >>> 16);
}
```
再散列的目的是减少散列冲突，使元素能够均匀地分布在不同的Segment中，从而提高容器的存取效率。

Segment中的`put`方法：
```java
// Segment 中的 put 方法
final V put(K key, int hash, V value, boolean onlyIfAbsent) {
    HashEntry<K,V> node = tryLock() ? null :
        scanAndLockForPut(key, hash, value);
    V oldValue;
    try {
        HashEntry<K,V>[] tab = table;
        int index = (tab.length - 1) & hash;
        HashEntry<K,V> first = entryAt(tab, index);
        for (HashEntry<K,V> e = first;;) {
            if (e != null) {
                K k;
                if ((k = e.key) == key ||
                    (e.hash == hash && key.equals(k))) {
                    oldValue = e.value;
                    if (!onlyIfAbsent) {
                        e.value = value;
                        ++modCount;
                    }
                    break;
                }
                e = e.next;
            }
            else {
                if (node != null)
                    node.setNext(first);
                else
                    node = new HashEntry<K,V>(hash, key, value, first);
                int c = count + 1;
                if (c > threshold && tab.length < MAXIMUM_CAPACITY)
                    rehash(node);
                else
                    setEntryAt(tab, index, node);
                ++modCount;
                count = c;
                oldValue = null;
                break;
            }
        }
    } finally {
        unlock();
    }
    return oldValue;
}
```
## 参考资料
* [我就知道面试官接下来要问我 ConcurrentHashMap 底层原理了](https://mp.weixin.qq.com/s/My4P_BBXDnAGX1gh630ZKw)
* [java中右移运算符>>和无符号右移运算符>>>的区别](https://blog.csdn.net/cobbwho/article/details/54907203)