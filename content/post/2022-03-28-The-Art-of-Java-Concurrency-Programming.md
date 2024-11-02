---
layout:     post

title:      "Java并发编程的艺术笔记"
# subtitle:   "Java并发编程的艺术笔记"
# excerpt: "Java并发编程的艺术笔记"
author:     "谢文进"
date:       2022-03-28
# description: "Java并发编程的艺术笔记"
image: "/img/2022-03-28-The-Art-of-Java-Concurrency-Programming/background.jpg"
published: 2022-03-28 
tags:
    - Java
    - 并发 

categories: [ Tech ]
URL: "/2022/03/28/The-Art-of-Java-Concurrency-Programming/"
---

# 第2章 Java并发机制的底层实现原理
## 2.1 volatile的应用
volatile的两条实现原则：
* Lock前缀指令会引起处理器缓存回写到内存。
* 一个处理器的缓存回写到内存会导致其他处理器的缓存无效。

## 2.2 synchronized的实现原理与应用
synchronized实现同步的基础：Java中的每一个对象都可以作为锁。具体表现为以下3种形式：
* 对于普通同步方法，锁是当前实例对象。
* 对于静态同步方法，锁是当前类的Class对象。
* 对于同步方法块，锁是Synchonized括号里配置的对象。

Synchonized在JVM里的实现，JVM基于进入和退出Monitor对象来实现方法同步和代码块同步。代码块同步是使用monitorenter和monitorexit指令实现的。

monitorenter指令在编译后插入到同步代码块的开始位置，而monitorexit是插入到方法结束处和异常处，JVM保证每个monitorenter必须有对应的monitorexit与之配对。

### 2.2.1 Java对象头

# 第3章 Java内存模型

## 3.1 Java内存模型的基础

### 3.1.1 并发编程模型的两个关键问题

* 线程之间如何通信
* 线程之间如何同步，同步是指程序中用于控制不同线程间操作发生相对顺序的机制。

线程之间的通信机制：共享内存和消息传递。

**Java的并发采用的是共享内存模型**，Java线程之间的通信总是隐式进行，整个通信过程对程序员完全透明。

### 3.1.2 Java内存模型的抽象结构

在Java中，所有实例域、静态域和数组元素都存储在堆内存中，堆内存在线程之间共享。

JMM定义了线程和主内存之间的抽象关系：**线程之间的共享变量存储在主内存中，每个线程都有一个私有的本地内存，本地内存中存储了该线程以读/写共享变量的副本。**

### 3.1.3 从源代码到指令序列的重排序

重排序分为3种：

1. 编译器优化的重排序
2. 指令级并行的重排序
3. 内存系统的重排序

### 3.1.4 并发编程模型的分类

由于写缓冲区仅对自己的处理器可见，它会导致处理器执行内存操作的顺序可能会与内存实际的操作执行顺序不一致。

常见的处理器都允许Store-Load重排序，常见的处理器都不允许对存在数据依赖的操作做重排序。

### 3.1.5 happens-before简介

happens-before仅仅要求前一个操作（执行的结果）对后一个操作可见，且前一个操作按顺序排在第二个操作之前。

一个happens-before规则对应于一个或多个编译器和处理器重排序规则。

## 3.2 重排序
### 3.2.1 数据依赖性
如果两个操作访问同一个变量，且这两个操作中有一个为写操作，此时这两个操作之间就存在数据依赖性。

编译器和处理器不会改变存在数据依赖关系的两个操作的执行顺序。

### 3.2.2 as-if-serial语义
as-if-serial语义：不管怎么重排序（编译器和处理器为了提高并行度），（单线程）程序的执行结果不能被改变。

### 3.2.3 程序顺序规则
A happens-before B，当实际执行时B却可以排在A之前执行。
软件技术和硬件技术的共同目标：在不改变程序执行结果的前提下，尽可能提高并行度。

### 3.2.4 重排序对多线程的影响
当代码中存在控制依赖时，会影响指令序列执行的并行度。为此，编译器和处理器会采用猜测执行来控制相关性对并行度的影响。

在多线程程序中，对存在控制依赖的操作重排序，可能会改变程序的执行结果。

## 3.3 顺序一致性
### 3.3.1 数据竞争与顺序一致性
如果程序是正确同步的，程序的执行将具有顺序一致性。

### 3.3.2 顺序一致性内存模型
顺序一致性内存模型有两大特性：
1. 一个线程中的所有操作必须按照程序的顺序来执行。
2. （不管程序是否同步）所有线程都只能看到一个单一的操作执行顺序。在顺序一致性内存模型中，每个操作都必须原子执行且立刻对所有线程可见。

未同步程序在JMM中不但整体的执行顺序是无序的，而且所有线程看到的操作执行顺序也可能不一致。

### 3.3.3 同步程序的顺序一致性结果
在JMM中，临界区内的代码可以重排序（但JMM不允许临界区内的代码“逸出”到临界区之外，那样会破坏监视器的语义）。

### 3.3.4 未同步程序的执行特性
对于未同步或未正确同步的多线程程序，JMM只提供最小安全性：线程执行时读取到的值，要么是之前某个线程写入的值，要么是默认值（0，Null，False），JMM保证线程读操作读取到的值不会无中生有的冒出来。

JMM不保证对64位的long型和double型变量的写操作具有原子性，而顺序一致性模型保证对所有的内存读/写操作都具有原子性。

从JSR-133内存模型开始（即从JDK5开始），仅仅只允许把一个64位long/double型变量的写操作拆分为两个32位的写操作来执行，任意的读操作在JSR-133中都必须具有原子性（即任意读操作必须要在单个读事务中执行）。

## 3.4 volatile的内存语义
### 3.4.1 volatile的特性
把对volatile变量的单个读/写，看成是使用同一个锁对这些单个读/写操作做了同步。
* 可见性。对一个volatile变量的读，总是能看到（任意线程）对这个volatile变量最后的写入。
* 原子性：对任意单个volatile变量的读/写具有原子性，但类似于volatile++这种复合操作不具有原子性。

### 3.4.2 volatile写-读建立的happens-before关系
volatile写和锁的释放有相同的内存语义；volatile读与锁的获取有相同的内存语义。

### 3.4.3 volatile写-读的内存语义
当写一个volatile变量时，JMM会把该线程对应的本地内存中的共享变量值刷新到主内存。
当读一个volatile变量时，JMM会把该线程对应的本地内存置为无效。线程接下来将从主内存中读取共享变量。

可以看作是在两个线程之间在进行消息传递。

### 3.4.4 volatile内存语义的实现
* 当第二个操作是volatile写时，不管第一个操作是什么，都不能重排序。
* 当第一个操作时volatile读时，不管第二个操作是什么，都不能重排序。
* 当第一个操作时volatile写，第二个操作是volatile读时，不能重排序。

基于保守策略的JMM内存屏障插入策略：
* 在每个volatile写操作的前面插入一个StoreStore屏障。
* 在每个volatile写操作的后面插入一个StoreLoad屏障。
* 在每个volatile读操作的后面插入一个LoadLoad屏障。
* 在每个volatile读操作的后面插入一个LoadStore屏障。

因为x86仅会对写-读操作做重排序，因此JMM仅需在volatile写操作的后面插入一个StoreLoad屏障。

### 3.4.5 JSR-133为什么要增强volatile的内存语义
严格限制编译器和处理器对volatile变量与普通变量的重排序，确保volatile的写-读和锁的释放-获取具有相同的内存语义。

在功能上，锁比volatile更强大；在可伸缩性和执行性能上，volatile更有优势。

## 3.5 锁的内存语义
### 3.5.1 锁的释放-获取建立的happens-before关系
锁除了让临界区互斥执行外，还可以让释放锁的线程向获取同一个锁的线程发送消息。

### 3.5.2 锁的释放和获取的内存语义
看作是两个线程之间通过主内存发送消息。

### 3.5.3 锁内存语义的实现
ReentrantLock的实现依赖于Java同步框架AbstractQueuedSynchronizer（简称AQS）。AQS使用一个整型的volatile变量（命名为state）来维持同步状态。

ReentrantLock分为公平锁和非公平锁。

使用公平锁时，加锁方法lock调用轨迹如下：
1. ReentrantLock:lock()
```java
public ReentrantLock() {
    sync = new NonfairSync(); // 默认构造器为非公平锁
}

public ReentrantLock(boolean fair) {
    sync = fair ? new FairSync() : new NonfairSync(); //true 为公平锁， false 为非公平锁
}

public void lock() {
    sync.lock(); // 如果为公平锁就调用FairSync的lock()，反之就调用NonfairSync.lock()
}
```

2. FairSync:lock() (ReentrantLock的内部类)
```java
final void lock() {
    acquire(1);
}
```

3. AbstractQueuedSynchronizer:acquire(int arg)
```java
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```

4. ReentrantLock:tryAcquire(int acquires)
```java
static final class FairSync extends Sync {
    private static final long serialVersionUID = -3000897897090466540L;

    final void lock() {
        acquire(1);
    }

    /**
        * Fair version of tryAcquire.  Don't grant access unless
        * recursive call or no waiters or is first.
        */
    protected final boolean tryAcquire(int acquires) {
        final Thread current = Thread.currentThread();
        /*
        在AQS中有如下的变量 state 以及 getSatate()方法：
        private volatile int state;
        protected final int getState() {
            return state;
        }
        */
        int c = getState(); // 获取锁的开始，首先读 volatile 变量 state
        if (c == 0) {
            if (!hasQueuedPredecessors() &&
                compareAndSetState(0, acquires)) {
                setExclusiveOwnerThread(current);
                return true;
            }
        }
        else if (current == getExclusiveOwnerThread()) {
            int nextc = c + acquires;
            if (nextc < 0)
                throw new Error("Maximum lock count exceeded");
            setState(nextc);
            return true;
        }
        return false;
}
```

使用公平锁时，解锁方法unlock()调用轨迹：
1. ReentrantLock:unlock()
```java
public void unlock() {
    sync.release(1);
}
```
2. AbstractQueuedSynchronizer:release(int arg)
```java
public final boolean release(int arg) {
    if (tryRelease(arg)) {
        Node h = head;
        if (h != null && h.waitStatus != 0)
            unparkSuccessor(h);
        return true;
    }
    return false;
}
```
3. Sync:tryRelase(int releases)
```java
protected final boolean tryRelease(int releases) {
    int c = getState() - releases;
    if (Thread.currentThread() != getExclusiveOwnerThread())
        throw new IllegalMonitorStateException();
    boolean free = false;
    if (c == 0) {
        free = true;
        setExclusiveOwnerThread(null);
    }
    /*
    AQS中的setState方法：
    protected final void setState(int newState) {
        state = newState;
    }
    */
    setState(c); // 释放锁的最后，写 volatile变量state
    return free;
}
```
公平锁在释放锁的最后写volatile变量state，在获取锁时首先读这个volatile变量。因此，释放锁的线程在写volatile变量之前可见的共享变量，在获取锁的线程读取同一个volatile变量后将立即变得对获取锁的线程可见。

非公平锁获取锁的方式和公平锁相同。非公平锁的加锁方法lock()调用轨迹如下：
1. ReentrantLock:lock()
2. NonfairSync:lock()
```java
final void lock() {
    if (compareAndSetState(0, 1))
        setExclusiveOwnerThread(Thread.currentThread());
    else
        acquire(1);
}
```

3. AbstractQueuedSynchronizer:compareAndSetState(int expect, int update)
```java
protected final boolean compareAndSetState(int expect, int update) {
    // See below for intrinsics setup to support this
    return unsafe.compareAndSwapInt(this, stateOffset, expect, update);
}
```
该方法以原子操作的方式更新state变量。compareAndSetState()简称CAS方法。如果当前状态值等于预期值，则以原子的方式将同步状态设置为给定的更新值。此操作具有volatile读和写的内存语义。

sun.misc.Unsafe类的compareAndSwapInt()方法的源代码：
```java
public final native boolean compareAndSwapInt(Object var1, long var2, int var4, int var5);
```
这是一个本地调用方法，会调用相应的C++代码，重要的是源代码中含有`cmpxchg`，程序会根据当前处理器的类型来决定是否为cmpxchg指令添加lock前缀。如果是当处理器，就省略lock指令，如果是多处理器就加上lock前缀。

intel手册对lock前缀的说明：
1. 确保对内存的读-改-写操作的原子执行。Intel使用缓存锁定来保证指令执行的原子性。缓存锁定将大大降低lock前缀指令的执行开销。
2. 禁止该指令，与之前和之后的读和写指令重排序。
3. 把写缓冲区中的所有数据刷新到内存中。

公平锁和非公平锁的内存语义：
* 公平锁和非公平锁释放时，最后都要写一个volatile变量state。
* 公平锁获取时，首先会去读volatile变量。
* 非公平锁获取时，首先会用CAS更新volatile变量，这个操作同时具有volatile读和volatile写的内存语义。

锁释放-获取的内存语义的实现至少有以下两种方式：
* 利用volatile变量的写-读所具有的内存语义。
* 利用CAS所附带的volatile读和volatile写的内存语义。

### 3.5.4 concurrent包的实现
一个通用化的实现模式：
* 首先，声明共享变量volatile.
* 然后，使用CAS的原子条件更新来实现线程之间的同步。
* 同时，配合以vilatile的读/写和CAS所具有的volatile读和写的内存语义来实现线程之间的同步。

## 3.6 final域的内存语义

### 3.6.1 final域的重排序规则

1. 在构造函数内对一个final域的写入，与随后把这个被构造对象的引用赋值给一个引用变量，这两个操作不能重排序。
2. 初次读一个包含final域的对象的引用，与随后初次读这个final域，这两个操作之间不能重排序。

### 3.6.2 写final域的重排序规则

写final域的重排序规则禁止把final域的写重排序到构造函数之外，具体实现：

1. JMM禁止编译器把final域的写重排序到构造函数之外。
2. 编译器会在final域的写之后，构造函数return之前，**插入一个StoreStore屏障**。这个屏障禁止处理器把final域的写重排序到构造函数之外。

写final域的重排序可以保证：在对象引用为任意线程可见之前，对象的final域已经被正确初始化过了，而普通域不具有这个保障。

### 3.6.3 读final域的重排序规则

在一个线程中，初次读对象引用与初次读该对象包含的final域，这两个操作之间不能重排序。实现：

1. JMM禁止处理器重排序这两个操作。
2. 编译器会在读final域操作的前面**插入一个LoadLoad屏障**。

这两个操作之间存在间接依赖关系。

读final域的重排序规则可以确保：在读一个对象的final域之前，一定会先读包含这个final域的对象的引用。

### 3.6.4 final域为引用类型

对于引用类型，写final域的重排序规则对编译器和处理器增加了如下约束：在构造函数内对一个final引用的对象的成员域的写入，与随后在构造函数外把这个被构造对象的引用赋值给一个引用变量，这两个操作之间不能重排序。

### 3.6.5 为什么final引用不能从构造函数内“溢出”

在构造函数返回前，被构造对象的引用不能为其他线程所见，因为此时的final域可能还没有被初始化。在构造函数返回后，任意线程都将保证能看到final域正确初始化之后的值。

### 3.6.6 final语义在处理器中的实现

X86处理器，只会对Store-Load进行重排序。根据final域重排序规则，以及X86处理器不会对存在间接依赖关系的操作进行重排序，因此在X86处理器中，final域的读/写不会插入任何内存屏障！

### 3.6.7 JSR-133为什么要增强final的语义

通过为final域增加写和读重排序规则，可以为Java程序员提供初始化的安全保证：只要对象是正确构造的（被构造对象的引用在构造函数中没有“溢出”），那么不需要使用同步（指lock和volatile的使用）就可以保证任意线程都能看到这个final域在构造函数中被初始化之后的值。

## 3.7 happens-before

### 3.7.1 JMM的设计

JSR-133专家组在设计JMM时的核心目标是找到一个好的平衡点：一方面，要为程序员提供足够强的内存可见性保证；另一方面，对编译器和处理器的限制尽可能地放松。

JMM对不同性质的重排序，采取不同的策略：

* 对于会改变程序执行结果的重排序，JMM要求编译器和处理器必须禁止这种重排序。
* 对于不会改变程序执行结果的重排序，JMM对编译器和处理器不做要求（JMM运行这种重排序）。

### 3.7.2 happens-before的定义

happens-before关系本质上和as-if-serial语义是一回事，都是为了在不改变程序执行结果的前提下，尽可能地提高程序执行的并行度。

### 3.7.3 happens-before规则

1. 程序顺序规则
2. 监视器锁规则
3. volatile变量规则
4. 传递性
5. start()规则：如果线程A执行操作`ThreadB.start()`（启动线程B），那么A线程的`ThreadB.start()`操作happens-before于线程B中的任意操作。
6. join()规则：如果线程A执行操作`ThreadB.join()`并成功返回，那么B线程中的任意操作happens-before于线程A从`ThreadB.join()`操作成功返回。

## 3.8 双重检查锁定与延迟初始化

### 3.8.1 双重检查锁定的由来

希望通过双重检查锁定来降低同步的开销。

### 3.8.2 问题的根源

创建对象`instance = new Instance();`可以分解为如下3行伪代码：

```
memory = allocate();		// 1:分配对象的内存空间
ctorInstance(memory);		// 2:初始化对象
instance = memory;     		// 3:设置instance指向刚分配的内存地址
```

2和3之间，可能被重排序，这会导致问题的发生。

两个办法实现线程安全的延迟初始化：

1. 不允许2和3重排序。
2. 允许2和3重排序，但不允许其他线程“看到”这个重排序。

### 3.8.3 基于volatile的解决方案

在双重检查锁定方法的基础上将instance声明为volatile型。

```java
private volatile static Instance instance;
```

2和3之间的重排序在多线程环境中会被禁止。

### 3.8.4 基于类初始化的解决方案

待读。

## 3.9 Java内存模型综述

### 3.9.1 处理器的内存模型

由于常见的处理器内存模型比JMM要弱，Java编译器在生成字节码时，会在执行指令序列的适当位置插入内存屏障来限制处理器的重排序。JMM在不同的处理器中需要插入的内存屏障的数量和种类也不相同。

### 3.9.2 各种内存模型之间的关系

JMM是一个语言级的内存模型，处理器内存模型是硬件级的内存模型，顺序一致性内存模型是一个理论参考模型。

### 3.9.3 JMM的内存可见性保证

三类：

* 单线程程序。
* 正确同步的多线程程序。
* 未同步/未正确同步的多线程程序。提供最小安全保障：线程执行时读取到的值，要么是之前某个线程写入的值，要么是默认值（0、null、false）。

最小安全保障与64位数据的非原子性写并不矛盾。最小安全性并不保证线程读取到的值，一定是某个线程写完后的值。最小安全性保证线程读取到的值不会是无中生有的冒出来，但并不保证线程读取到的值一定是正确的。

### 3.9.4 JSR-133对旧内存模型的修补

主要有两个：

* 增强volatile的内存语义。
* 增强final的内存语义。

# 第4章 Java并发编程基础

## 4.1 线程简介

### 4.1.1 什么是线程

现代操作系统调度的最小单元是线程，也叫轻量级进程，在一个进程里可以创建多个线程，这些线程都拥有各自的计数器、堆栈和局部变量等属性，并且能够访问共享的内存变量。

Java程序天生就是多线程程序，因为执行main()方法的是一个名称为main的线程。

### 4.1.2 为什么要使用多线程

* 更多的处理器核心
* 更快的响应时间
* 更好的编程模型

### 4.1.3 线程优先级

在Java线程中，通过一个整型成员变量priority来控制优先级，优先级的范围从1～10，在线程构建的时候可以通过`setPriority(int)`方法来修改优先级，默认优先级是5，优先级高的线程分配时间片的数量要多于优先级低的线程。

程序正确性不能依赖线程的优先级高低，有些操作系统会忽略对线程优先级的设定。

### 4.1.4 线程的状态

![LICENSE](/img/2022-03-28-The-Art-of-Java-Concurrency-Programming/LICENSE.png)

线程在执行Runnable的`run()`方法之后将会进入终止状态。

### 4.1.5 Daemon线程

Daemon线程是一种支持型线程，因为它主要被用做程序中后台调度及支持性工作，也就是守护线程。当一个Java虚拟机中不存在非Daemon线程的时候，Java虚拟机将会退出。

调用`Thread.setDaemon(true)`将线程设置为Daemon线程。

在Java虚拟机退出时Daemon线程中的finally块不一定会执行。

## 4.2 启动和终止线程

### 4.2.1 构造线程

构造一个线程，会调用Thread.java中的构造器：

```java
public Thread(Runnable target, String name) {
    init(null, target, name, 0);
}
```

接着调用init方法：

```java
private void init(ThreadGroup g, Runnable target, String name,
                  long stackSize) {
    init(g, target, name, stackSize, null, true);
}
```

调用下面的init方法：

```java
private void init(ThreadGroup g, Runnable target, String name,
                  long stackSize, AccessControlContext acc,
                  boolean inheritThreadLocals) {
    if (name == null) {
        throw new NullPointerException("name cannot be null");
    }

    this.name = name;

  	// 当前线程就是该线程的父线程
    Thread parent = currentThread();
    SecurityManager security = System.getSecurityManager();
    if (g == null) {
        /* Determine if it's an applet or not */

        /* If there is a security manager, ask the security manager
           what to do. */
        if (security != null) {
            g = security.getThreadGroup();
        }

        /* If the security doesn't have a strong opinion of the matter
           use the parent thread group. */
        if (g == null) {
            g = parent.getThreadGroup();
        }
    }

    /* checkAccess regardless of whether or not threadgroup is
       explicitly passed in. */
    g.checkAccess();

    /*
     * Do we have the required permissions?
     */
    if (security != null) {
        if (isCCLOverridden(getClass())) {
            security.checkPermission(SUBCLASS_IMPLEMENTATION_PERMISSION);
        }
    }

    g.addUnstarted();

    this.group = g;
  	// 将 daemon、priority属性设置为父线程的对应属性
    this.daemon = parent.isDaemon();
    this.priority = parent.getPriority();
    if (security == null || isCCLOverridden(parent.getClass()))
        this.contextClassLoader = parent.getContextClassLoader();
    else
        this.contextClassLoader = parent.contextClassLoader;
    this.inheritedAccessControlContext =
            acc != null ? acc : AccessController.getContext();
    this.target = target;
    setPriority(priority);
  	// 将父线程的 inheritThreadLocal 复制过来
    if (inheritThreadLocals && parent.inheritableThreadLocals != null)
        this.inheritableThreadLocals =
            ThreadLocal.createInheritedMap(parent.inheritableThreadLocals);
    /* Stash the specified stack size in case the VM cares */
    this.stackSize = stackSize;

    /* Set thread ID */
    tid = nextThreadID(); // 分配一个线程ID
}
```

一个新构造的线程对象是由其parent线程来进行空间分配的，而child线程继承了parent是否为Daemon、优先级和加载资源的contextClassLoader以及可继承的ThreadLocal，同时还会分配一个唯一的ID来标识这个child线程。

### 4.2.2 启动线程

线程strart()方法的含义：当前线程（即parent线程）同步告知Java虚拟机，只要线程规划器空闲，应立即启动调用start()方法的线程。

### 4.2.3 理解中断

中断可以理解为线程的一个标识位属性，它表示一个运行中的线程是否被其他线程进行了中断操作。

线程通过检查自身是否被中断来进行响应，线程通过方法`isInterrupted()`来进行判断是否被中断。

```java
public boolean isInterrupted() {
    return isInterrupted(false);
}

private native boolean isInterrupted(boolean ClearInterrupted);
```

或者调用静态方法`Thread.interrupted()`对当前线程的中断标识位进行复位。

```java
public static boolean interrupted() {
        return currentThread().isInterrupted(true);
}
```

如果该线程已经处于终结状态，即使该线程被中断过，在调用该线程对象的`isInterrupted()`时依旧会返回false。

许多方法在抛出InterruptedException之前，Java虚拟机会先将该线程的中断标识位清除，然后抛出InterruptedException，此时调用isInterrupted()方法将会返回false。

```java
 /**
 * @throws  InterruptedException
 *          if any thread has interrupted the current thread. The
 *          <i>interrupted status</i> of the current thread is
 *          cleared when this exception is thrown.
 */
public static native void sleep(long millis) throws InterruptedException;
```

### 4.2.4 过期的suspend()、resume()和stop()

suspend()、resume()和stop()类似CD机播放音乐时的暂停、恢复和停止操作。

### 4.2.5 安全地终止线程

* 中断操作
* 利用一个boolean变量来控制是否需要停止任务并终止该线程。

这两种方法可以使线程在终止时有机会去清理资源。

## 4.3 线程间通信
### 4.3.1 volatile和synchronized关键字
* Java支持多个线程同时访问一个对象或者对象的成员变量，由于每个线程可以拥有这个变量的拷贝，所以程序在执行过程中，一个线程看到的变量不一定是最新的。
* 关键字volatile可以用来修饰字段（成员变量）。
* 关键字synchronized可以修饰方法或者以同步块的形式来进行使用。
* 通过class信息，对于同步块的实现使用了monitorenter和monitorexit指令，而同步方法则是依靠方法修饰符上的ACC_SYNCHRONIZED来完成。
* 无论采用哪种方式，其本质是对一个对象的监视器进行获取，而这个获取过程是排他的，也就是同一时刻只能有一个线程获取到由synchronized所保护对象的监视器。
* 任意一个对象都拥有自己的监视器。
* 对象、监视器、同步队列和执行线程之间的关系图

### 4.3.2 等待/通知机制
* 等待/通知的相关方法是任意Java对象都具备的，被定义在超类java.lang.Object上。
* 等待/通知机制，是指一个线程A调用了对象O的wait()方法进入等待状态，而另一个线程B调用了对象O的notify()或者notifyAll()方法，线程A收到通知后从对象O的wait()方法返回，进而执行后续操作。

细节：
1. 使用wait()、notify()和notifyAll()时需要先对调用对象加锁。
2. 调用wait()方法后，线程状态由RUNNING变为WAITING，并将当前线程放置到对象的等待队列。
3. notify()或notifyAll()方法调用后，等待线程依旧不会从wait()返回，需要调用notify()或notifyAll()的线程释放锁之后，等待线程才有机会从wait()返回。
4. notify()方法将等待队列中的一个等待线程从等待队列中移到同步队列中，而notifyAll()方法则是将等待队列中所有的线程全部移到同步队列，被移动的线程状态由WAITING变为BLOCKED。
5. 从wait()方法返回的前提是获得了调用对象的锁。

### 4.3.3 等待/通知的经典范式
等待方遵循如下原则：
   1. 获取对象的锁。
   2. 如果条件不满足，那么调用对象的wait()方法，被通知后仍要检查条件。
   3. 条件满足则执行对应的逻辑。

通知方遵循如下原则：
   1. 获得对象的锁。
   2. 改变条件。
   3. 通知所有等待在对象上的线程。

### 4.3.4 管道输入/输出流
* 管道输入/输出流主要用于线程之间的数据传输，而传输的媒介为内存。
* 4种具体实现：PipedOutputStream、PipedOutputStream、PipedReader和PipedWriter，前两种面向字节，后两种面向字符。
* 对于Piped类型的流，必须先要进行绑定，也就是调用connect()方法，如果没有将输入/输出流绑定起来，对于该流的访问将会抛出异常。

### 4.3.5 Thread.join()的使用
* 如果一个线程A执行了thread.join()语句，其含义是：当线程A等待thread线程终止之后才从thread.join()返回。
* 每个线程终止的前提是前驱线程终止，每个线程等待前驱线程终止后，才从join()方法返回，这里涉及了等待/通知机制。
* 当线程终止时，会调用线程自身的notifyAll()方法，会通知所有等待在该线程对象上的线程。

### 4.3.6 ThreadLocal的使用
* ThreadLocal，即线程变量，是一个以ThreadLocal对象为键、任意对象为值的存储结构，这个结构被附带在线程上，也就是说一个线程可以根据一个ThreadLocal对象查询到绑定在这个线程上的一个值。
* 可以通过set(T)方法来设置一个值，在当前线程下再通过get()方法获取到原先设置的值。