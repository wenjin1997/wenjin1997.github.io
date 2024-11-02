---
layout:     post

title:        "Rust异步编程"
# subtitle:   "Java并发编程的艺术笔记"
# excerpt:    "Java并发编程的艺术笔记"
author:       "谢文进"
date:         2022-07-07
description:  "记录学习Rust异步编程的笔记。"
# image:      "/img/2022-03-28-The-Art-of-Java-Concurrency-Programming/background.jpg"
published:    2022-07-07 
tags:
     - Rust
     - 异步

categories:   [ Tech ]
URL:          "/2022/07/07/Rust-async/"
---

本章至下而上的方式来带领大家理解异步编程:

1. 异步 I/O 模型
2. 异步编程模型：
   - 事件驱动模型
   - Futures
   - 生成器 与 Pin
   - async/await
   - 异步运行时介绍：async-std、tokio、bastion、smol

## 异步IO模型

### 基本概念

- 同步和异步，关注的是消息通信机制。（调用者视角）结合烧开水的例子。
  - 同步，发出一个调用，在没有得到结果之前不返回。水没烧好不离开。
  - 异步，发出一个调用，在没有得到结果之前返回。水没烧好时，可以先去干别的。
- 阻塞和非阻塞，关注的是程序等待调用结果的状态。（被调用者视角）
  - 阻塞，在调用结果返回之前，线程被挂起。
  - 非阻塞，在调用结果返回之前，线程不会被挂起。

阻塞，与系统调用有关。

### I/O模型

```text
                                 +-+ 阻 塞 I/O (BIO)
                                 |
                                 +-+ 非 阻 塞 I/O (NIO)
                                 |
              +----+ 同 步 I/O +--+
              |                  |
              |                  +-+ I/O 多 路 复 用
              |                  |
              |                  +-+ 信 号 驱 动 I/O
I/O 模 型  +---+
              |
              |
              |                  +-+ Linux (AIO)
              |                  |         (io_uring)
              +----+ 异 步 I/O +--+
                                 |
                                 +-+ windows (IOCP)
```

### 同步阻塞I/O

```text
Application               kernel
+---------+            +-----------+  +---+
|         |   syscall  | no        |      |
|   Read  | +--------> | datagram  |      |
| recvfrom|            | ready     |      |
|         |            |    +      |      +-+ wait for
|         |            |    |      |      +-+ data
|         |            |    v      |      |
|         |            | datagram  |      |
|         |            | ready     |  +---+
|         |            |           |
|         |            | copy      |  +---+
|         |            | datagram  |      |
|process  |            |    +      |      +-+ copy data
|datagram |   return   |    |      |      +-+ from kernel to user
|         | <--------+ |    v      |      |
|         |            |  copy     |  +---+
|         |            |  complete |
+---------+            +-----------+
```

输入操作两个阶段：

1. 进程等待内核把数据准备好；这个阶段可以阻塞也可非阻塞，设置socket属性。
   - 阻塞： recvfrom 阻塞线程直到返回数据就绪的结果。
   - 非阻塞：立即返回一个错误，轮询直到数据就绪。
2. 从内核缓冲区向进程缓冲区复制数据。（一直阻塞）

异步I/O，recvfrom总是立即返回，两个阶段都由内核完成。

### I/O多路复用

IO多路复用是一种同步IO模型，实现一个线程可以监视多个文件句柄。

支持I/O多路复用的系统调用有 select/pselect/poll/epoll，本质都是 同步 I/O，因为数据拷贝都是阻塞的。 通过 select/epoll 来判断数据报是否准备好，即判断可读可写状态。

### 补充资料

I/O模型和I/O调用是不是分不清楚，参考[理解一下5种IO模型、阻塞IO和非阻塞IO、同步IO和异步IO](https://cloud.tencent.com/developer/article/1684951)。

先理解什么是IO：

![image-20220815091311443](/img/2022-07-07-Rust-async/image-20220815091311443.png)

要输入输出数据分为两个阶段：用户进程空间<-->内核空间、内核空间<-->设备空间（磁盘、网络等）。

1. 阻塞IO模型

   ![image-20220815091553220](/img/2022-07-07-Rust-async/image-20220815091553220.png)

   进程发起IO系统调用后，进程被阻塞，转到内核空间处理，整个IO处理完毕后返回进程。操作成功则进程获取到数据。

2. 非阻塞IO模型

   ![image-20220815091819969](/img/2022-07-07-Rust-async/image-20220815091819969.png)

   进程发起IO系统调用后，如果内核缓冲区没有数据，需要到IO设备中读取，进程返回一个错误而不会被阻塞；进程发起IO系统调用后，如果内核缓冲区有数据，内核就会把数据返回进程。

3. IO复用模型

   ![image-20220815092054045](/img/2022-07-07-Rust-async/image-20220815092054045.png)

   多个进程的IO可以注册到一个复用器（select）上，然后用一个进程调用该select，select会监听所有注册进来的IO；

   如果select没有监听的IO在内核缓冲区都没有可读数据，select调用进程会被阻塞；而当任一IO在内核缓冲区中有可用数据时，select调用就会返回；

   而后select调用进程可以自己或通知另外的进程（注册进程）来再次发起读取IO，读取内核中准备好的数据。

   select、poll、epoll

   * Linux中IO复用的实现方式主要有select、poll和epoll。
   * Select: 注册IO、阻塞扫描，监听的IO最大连接数不能多于FD_SIZE。
   * Poll: 原理和Select相似，没有数量限制，但IO数量大扫描线性性能下降。
   * Epoll：事件驱动不阻塞，mmap实现内核与用户空间的消息传递，数量很大，Linux2.6后内核支持。

4. 信号驱动IO模型

   ![image-20220815093027374](/img/2022-07-07-Rust-async/image-20220815093027374.png)

   当进程发起一个IO操作，会向内核注册一个信号处理函数，然后进程返回不阻塞；当内核数据就绪时会发生一个信号给进程，进程便在信号处理函数中调用IO读取数据。

   特点：回调机制，实现、开发应用难度大。

5. 异步IO模型

   ![image-20220815093324750](/img/2022-07-07-Rust-async/image-20220815093324750.png)

   当进程发起一个IO操作，进程返回（不阻塞），但也不能返回结果；内核把整个IO处理完后，会通知进程结果。如果IO操作成功则进程直接获取到数据。

一图总结：

![img](/img/2022-07-07-Rust-async/rust_async_I:O_model.png)

## epoll

```text
                        +--------------------------------+     +-------------------------+
                        | epoll_ctl                      |     | epoll_wait              |
                        |                                |     |                         |
                        |                                |     |         +----+          |
                        |                 +---+          |     |         |    |          |
                        |                 |   |          |     |         |    |          |
                        |               +-+---+--+       |     |         +--+-+          |
                        |               |        |       |     |            |            |
                        |            +--++     +-++      |     |            |            |
epoll_create  +---->    |            |   |     |  |      |     |         +--+-+          |
                        |            +-+-+     +--+      +---->+         |    |          |
                        |              |                 |event|         |    |          |
                        |         +----+--+              |     |         +--+-+          |
                        |         |       |              |     |            |            |
                        |         ++      |              |     |            |            |
                        |        +--+   +-+-+            |     |         +--+-+          |
                        |        |  |   |   |            |     |         |    |          |
                        |        +--+   +---+            |     |         |    |          |
                        |                                |     |         +----+          |
                        |                    红 黑 树     |     |                 链 表    |
                        +--------------------------------+     +-------------------------+
```

- epoll_create(int size) : 内核产生一个epoll实例数据结构，并返回一个epfd
- epoll_ctl(int epfd, int op, int fd, struct epoll_event *event)：将被监听的描述符添加到红黑树或从红黑树中删除或者对监听事件进行修改。
- epoll_wait(int epfd, struct epoll_event *events, int maxevents, int timeout): 阻塞等待注册的事件发生，返回事件的数目，并将触发的事件写入events数组中

epoll 两种触发机制：

- 水平触发机制（LT)。缓冲区只要有数据就触发读写。epoll 默认工作方式。select/poll只支持该方式。
- 边缘触发机制（ET)。缓冲区空或满的状态才触发读写。nginx 使用该方式，避免频繁读写。

惊群问题：

当多个进程/线程调用epoll_wait时会阻塞等待，当内核触发可读写事件，所有进程/线程都会进行响应，但是实际上只有一个进程/线程真实处理这些事件。 Liux 4.5 通过引入 EPOLLEXCLUSIVE 标识来保证一个事件发生时候只有一个线程会被唤醒，以避免多侦听下的惊群问题。

### 补充资料

[Linux下的I/O复用与epoll详解](https://www.cnblogs.com/lojunren/p/3856290.html)

select、poll、epoll

- Linux中IO复用的实现方式主要有select、poll和epoll：
- Select：注册IO、阻塞扫描，监听的IO最大连接数不能多于FD_SIZE；
- Poll：原理和Select相似，没有数量限制，但IO数量大扫描线性性能下降；
- Epoll ：事件驱动不阻塞，mmap实现内核与用户空间的消息传递，数量很大，Linux2.6后内核支持。

select的缺陷：

* select预估错误了一件事，当数十万并发连接存在时，可能每一毫秒只有数百个活跃的连接，同时其余数十万连接在这一毫秒是非活跃的。
* 在Linux内核中，select所用到的FD_SET是有限的，即内核中有个参数__FD_SETSIZE定义了每个FD_SET的句柄个数。
* 内核中实现 select是用**轮询**方法，即每次检测都会遍历所有FD_SET中的句柄，显然，select函数执行时间与FD_SET中的句柄个数有一个比例关系，即 select要检测的句柄数越多就会越费时。

相比于select机制，poll只是取消了最大监控文件描述符数限制，并没有从根本上解决select存在的问题。

epoll精巧的使用了3个方法来实现select方法要做的事：

1. 新建epoll描述符==epoll_create()
2. epoll_ctl(epoll描述符，添加或者删除所有待监控的连接)
3. 返回的活跃连接 ==epoll_wait（ epoll描述符 ）

与select相比，epoll分清了频繁调用和不频繁调用的操作。例如，epoll_ctl是不太频繁调用的，而epoll_wait是非常频繁调用的。这时，epoll_wait却几乎没有入参，这比select的效率高出一大截，而且，它也不会随着并发连接的增加使得入参越发多起来，导致内核执行效率下降。

 要深刻理解epoll，首先得了解epoll的三大关键要素：**mmap、红黑树、链表**。

epoll是通过内核与用户空间mmap同一块内存实现的。mmap将用户空间的一块地址和内核空间的一块地址同时映射到相同的一块物理内存地址（不管是用户空间还是内核空间都是虚拟地址，最终要通过地址映射映射到物理地址），使得这块物理内存对内核和对用户均可见，减少用户态和内核态之间的数据交换。内核可以直接看到epoll监听的句柄，效率高。

[红黑树](https://zh.m.wikipedia.org/zh/%E7%BA%A2%E9%BB%91%E6%A0%91)将存储epoll所监听的套接字。上面mmap出来的内存如何保存epoll所监听的套接字，必然也得有一套数据结构，epoll在实现上采用红黑树去存储所有套接字，当添加或者删除一个套接字时（epoll_ctl），都在红黑树上去处理，红黑树本身插入和删除性能比较好，时间复杂度O(logN)。

 通过epoll_ctl函数添加进来的事件都会被放在红黑树的某个节点内，所以，重复添加是没有用的。当把事件添加进来的时候时候会完成关键的一步，那就是该事件都会与相应的设备（网卡）驱动程序建立回调关系，当相应的事件发生后，就会调用这个回调函数，该回调函数在内核中被称为：ep_poll_callback,**这个回调函数其实就所把这个事件添加到rdllist这个双向链表中**。一旦有事件发生，epoll就会将该事件添加到双向链表中。那么当我们调用epoll_wait时，epoll_wait只需要检查rdlist双向链表中是否有存在注册的事件，效率非常可观。这里也需要将发生了的事件复制到用户态内存中即可。

![img](/img/2022-07-07-Rust-async/rust_async_epoll.jpeg)

  epoll_wait的工作流程：

1. epoll_wait调用ep_poll，当rdlist为空（无就绪fd）时挂起当前进程，直到rdlist不空时进程才被唤醒。
2. 文件fd状态改变（buffer由不可读变为可读或由不可写变为可写），导致相应fd上的回调函数ep_poll_callback()被调用。
3. ep_poll_callback将相应fd对应epitem加入rdlist，导致rdlist不空，进程被唤醒，epoll_wait得以继续执行。
4. ep_events_transfer函数将rdlist中的epitem拷贝到txlist中，并将rdlist清空。
5. ep_send_events函数（很关键），它扫描txlist中的每个epitem，调用其关联fd对用的poll方法。此时对poll的调用仅仅是取得fd上较新的events（防止之前events被更新），之后将取得的events和相应的fd发送到用户空间（封装在struct epoll_event，从epoll_wait返回）。  

select、poll和epoll三种I/O复用模式的比较：

| 系统调用                               | select                                                       | poll                                                         | epoll                                                        |
| -------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 事件集合                               | 用户通过3个参数分别传入感兴趣的可读，可写及异常等事件内核通过对这些参数的在线修改来反馈其中的就绪事件这使得用户每次调用select都要重置这3个参数 | 统一处理所有事件类型，因此只需要一个事件集参数。用户通过pollfd.events传入感兴趣的事件，内核通过修改pollfd.revents反馈其中就绪的事件 | 内核通过一个事件表直接管理用户感兴趣的所有事件。因此每次调用epoll_wait时，无需反复传入用户感兴趣的事件。epoll_wait系统调用的参数events仅用来反馈就绪的事件 |
| 应用程序索引就绪文件描述符的时间复杂度 | O(n)                                                         | O(n)                                                         | O(1)                                                         |
| 最大支持文件描述符数                   | 一般有最大值限制                                             | 65535                                                        | 65535                                                        |
| 工作模式                               | LT                                                           | LT                                                           | 支持ET高效模式                                               |
| 内核实现和工作效率                     | 采用轮询方式检测就绪事件，时间复杂度：O(n)                   | 采用轮询方式检测就绪事件，时间复杂度：O(n)                   | 采用回调方式检测就绪事件，时间复杂度：O(1)                   |

## io_uring 异步 I/O 模型

Linux AIO 实现的并不理想，所以引入了新的异步I/O接口 io_uring。

```text
+----+ Head  +---------+               +----------+ Head
|            |         |               |          |
|            |         |               |          |
|            +---------+               +----------+
|            |         |               |          |
|            |         |               |          |
|            +---------+               +----------+
|            |         |               |          |
|            |         |               |          |
|            +---------+               +----------+
|            |         |               |          |
|      Tail  +---------+               +----------+ Tail <--+
|        +--------------------------------------------+     |
|        | Kernel                                     |     |
|        |                                            |     |
|        |        +-------+              +-------+    |     |
|        |        |       |              |       |    |     |
+---------------> | SQ    |              |  CQ   | +--------+
         |        |       |              |       |    |
         |        +-------+              +-------+    |
         |                                            |
         +--------------------------------------------+
```

io_uring接口通过两个主要数据结构工作：

- 提交队列条目（sqe）
- 完成队列条目（cqe）

这些结构的实例位于内核和应用程序之间的**共享内存**单生产者单消费者环形缓冲区中。

参考：

* [How io_uring and eBPF Will Revolutionize Programming in Linux](https://thenewstack.io/how-io_uring-and-ebpf-will-revolutionize-programming-in-linux/)
* [A Universal I/O Abstraction for C++](https://cor3ntin.github.io/posts/iouring/#io_uring)

![image-20220815104458456](/img/2022-07-07-Rust-async/image-20220815104458456.png)

## 事件驱动编程模型

因为处理 I/O 复用的编程模型相当复杂，为了简化编程，引入了下面两种模型。

- Reactor（反应器） 模式，对应同步I/O，被动的事件分离和分发模型。服务等待请求事件的到来，再通过不受间断的同步处理事件，从而做出反应。
- Preactor（主动器） 模式，对应异步I/O，主动的事件分离和分发模型。这种设计允许多个任务并发的执行，从而提高吞吐量；并可执行耗时长的任务（各个任务间互不影响）。

Reactor Model:

```text
                                                     +----------------+
req                                        Dispatch  |                |
+------+                                  +--------> | req handler    |
|      |                                  |          +----------------+
|      | +----+                           |
+------+      | event    +------------+   |
              |          |            |   |
              +--------> |  Service   |   |Dispatch  +----------------+
                         |  Handler   +------------> |                |
req          +---------> |            |   |          | req handler    |
+------+     |           +------------+   |          +----------------+
|      |     | event                      |
|      +----+                             |
+------+                                  | Dispatch +----------------+
                                          +--------->+                |
                                                     | req handler    |
                                                     +----------------+
```

三种实现方式：

- 单线程模式。 accept()、read()、write()以及connect()操作 都在同一线程。
- 工作者线程池模式。非 I/O 操作交给线程池处理
- 多线程模式。主Reactor (master) ，负责网络监听 ， 子Reactor(worker) 读写网络数据。

读写操作流程：

1. 应用注册读写就绪事件和相关联的事件处理器
2. 事件分离器等待事件发生
3. 当发生读写就绪事件，事件分离器调用已注册的事件处理器
4. 事件处理器执行读写操作

参与者：

1. 描述符（handle）：操作系统提供的资源，识别 socket等。
2. 同步事件多路分离器。开启事件循环，等待事件的发生。封装了 多路复用函数 select/poll/epoll等。
3. 事件处理器。提供回调函数，用于描述与应用程序相关的某个事件的操作。
4. 具体的事件处理器。事件处理器接口的具体实现。使用描述符来识别事件和程序提供的服务。
5. Reactor 管理器。事件处理器的调度核心。分离每个事件，调度事件管理器，调用具体的函数处理某个事件。

### 补充资料

[事件驱动及其设计模式](https://segmentfault.com/a/1190000022048085)

从事件角度说，事件驱动程序的基本结构是由一个事件收集器、一个事件发送器和一个事件处理器组成。

**事件循环器的实现**

事件循环器不断接受来自客户端（Client）的请求，事件循环器把请求转交给注册了某类事件的工作线程（Worker）处理。

![image-20220815105703262](/img/2022-07-07-Rust-async/image-20220815105703262.png)

根据实现的方式不同，在网络编程中基于事件驱动主要有两种设计模式：Reactor和Proactor。

**Reactor模式框架**

使用Reactor模型，必备的几个组件：事件源、Reactor框架、事件多路复用机制和事件处理程序，先来看看Reactor模型的整体框架，接下来再对每个组件做逐一说明。

1）事件源：Linux上是文件描述符，Windows上就是Socket或者Handle了，这里统一称为“句柄集”；程序在指定的句柄上注册关心的事件，比如I/O事件。

2）事件多路复用机制：由操作系统提供的I/O多路复用机制，比如select和epoll。程序首先将其关心的句柄（事件源）及其事件注册到多路复用机制上。当有事件到达时，事件多路复用机制会发出通知“在已经注册的句柄集中，一个或多个句柄的事件已经就绪”。程序收到通知后，就可以在非阻塞的情况下对事件进行处理了。

3） Reactor。是事件管理的接口，内部使用事件多路复用机制注册、注销事件；并运行事件循环，当有事件进入“就绪”状态时，调用注册事件的回调函数处理事件。

4）事件处理程序。事件处理程序提供了一组接口，每个接口对应了一种类型的事件，供Reactor在相应的事件发生时调用，执行相应的事件处理。通常它会绑定一个有效的句柄。

使用Reactor模式后，事件控制流是什么样子呢？可以参见下面的序列图。

![img](/img/2022-07-07-Rust-async/rust_async_reactor.webp)

我们分别以读操作和写操作为例来看看Reactor中的具体步骤：

1) 应用程序注册读就绪事件和相关联的事件处理器；

2) 事件分离器等待事件的发生；

3) 当发生读就绪事件的时候，事件分离器调用第一步注册的事件处理器；

4) 事件处理器首先执行实际的读取操作，然后根据读取到的内容进行进一步的处理。

写入操作类似于读取操作，只不过第一步注册的是写就绪事件。

**Proactor**

我们来看看Proactor模式中读取操作和写入操作的过程：

1) 应用程序初始化一个异步读取操作，然后注册相应的事件处理器，此时事件处理器不关注读取就绪事件，而是关注读取完成事件，这是区别于Reactor的关键。

2) 事件分离器等待读取操作完成事件。

3) 在事件分离器等待读取操作完成的时候，操作系统调用内核线程完成读取操作（异步I/O都是操作系统负责将数据读写到应用传递进来的缓冲区供应用程序操作），并将读取的内容放入用户传递过来的缓存区中。这也是区别于Reactor的一点。

4) 事件分离器捕获到读取完成事件后，激活应用程序注册的事件处理器，事件处理器直接从缓存区读取数据，而不需要进行实际的读取操作。

Proactor中写入操作和读取操作，只不过感兴趣的事件是写入完成事件。

从上面可以看出，Reactor和Proactor模式的主要区别就是真正的读取和写入操作是由谁来完成的，Reactor中需要应用程序自己读取或者写入数据，而Proactor模式中，应用程序不需要进行实际的读写过程，它只需要从缓存区读取或者写入即可，操作系统会读取缓存区或者写入缓存区到真正的I/O设备。

## epoll代码实践

参见[rust-epoll-example](https://github.com/zupzup/rust-epoll-example/blob/main/src/main.rs)。

## Reactor代码实践

参见[rust-reactore-executor-example](https://github.com/zupzup/rust-reactor-executor-example)。

## MiniMio代码实践

epoll只支持在Linux系统下使用，而[minimio](https://github.com/cfsamson/examples-minimio)实现了跨平台。

会将各个平台的命令拿出来。比如Selector做了抽象，每个平台实际上不同。

```rust
#[cfg(target_os = "windows")]
mod windows;
#[cfg(target_os = "windows")]
pub use windows::{Event, Registrator, Selector, TcpStream};

#[cfg(target_os = "macos")]
mod macos;
#[cfg(target_os = "macos")]
pub use macos::{Event, Registrator, Selector, TcpStream};

#[cfg(target_os = "linux")]
mod linux;
#[cfg(target_os = "linux")]
pub use linux::{Event, Registrator, Selector, TcpStream};
```

在lib.rs中抽象出跨平台的结构Poll。代表事件队列。

```rust
#[derive(Debug)]
pub struct Poll {
    registry: Registry,
    is_poll_dead: Arc<AtomicBool>,
}
```

Poll有一个轮询方法poll。在实际调用时已经实现了跨平台。

```rust
pub fn poll(&mut self, events: &mut Events, timeout_ms: Option<i32>) -> io::Result<usize>{
  // ...
}
```

## Mio代码实践

生产环境中的多路复用跨平台代码[mio](https://github.com/tokio-rs/mio)。

其中抽象出了[Poll](https://docs.rs/mio/0.8.4/mio/struct.Poll.html)结构体。

```rust
pub struct Poll {
    registry: Registry,
}
```

这里用到了设计模式——外观模式，外观模式相关知识见[这里](https://www.runoob.com/design-pattern/facade-pattern.html)。

外观模式（Facade Pattern）隐藏系统的复杂性，并向客户端提供了一个客户端可以访问系统的接口。这种类型的设计模式属于结构型模式，它向现有的系统添加一个接口，来隐藏系统的复杂性。

这种模式涉及到一个单一的类，该类提供了客户端请求的简化方法和对现有系统类方法的委托调用。

## 异步编程模型概要

让我们先从建立异步编程模型的整体概念框架开始，先不深入细节。

**Rust 提供的异步并发相比于其他语言有什么特点？**

1. Rust 语言只是提供一个零成本的异步编程抽象，而不内置运行时。
2. 基于 Generator 实现的 Future，在 Future 基础上 提供 async/await 语法糖。本质是一个状态机。

查看 README 和其他编程语言比较的图示。

![node](/img/2022-07-07-Rust-async/node_async.png)

![go](/img/2022-07-07-Rust-async/go_async.png)

![rust](/img/2022-07-07-Rust-async/rust_async_01.png)

![rustasync](/img/2022-07-07-Rust-async/async_01.png)

**为什么需要异步？**

1. 对极致性能的追求。
2. 对编程体验的追求。

**异步编程模型发展阶段：**

1. Callback
2. Promise/Future
3. async/await

可在项目 README 查看回调地狱示例图。

![callback_hell](/img/2022-07-07-Rust-async/callback-hell.gif)

```text
A
+------+
|      |
|      |
|      +-------------------+
|      |                   |
|      |                   |
|      |                  Bv
+------+                +-----+
|      |                |     |
|      |                | do1 |
|      |                |     |
|      |                +-----+
|      |                |     |
|      |                | do2 |
|      |                +-+---+
|      |                  |
|      |                  |
| A    |                  |
+------+                  |
|      |                  |
|      |                  |
|      |                  |
|      | <----------------+
|      |
|      |
|      |
+------+
```

早期 Rust 异步写法示意：

```rust
let future = id_rpc(&my_server).and_then(|id| {
    get_row(id)
}).map(|row| {
    json::encode(row)
}).and_then(|encoded| {
    write_string(my_socket, encoded)
});
```

这样写会存在大量内嵌 Future，开发体验不好。

引入 async/await 之后：

```rust
let id = id_rpc(&my_server).await;
let row = get_row(id).await;
let encoded = json::encode(row);
write_string(my_socket, encoded).await;
```

拥有了和同步代码一致的体验。

**异步任务可看作是一种绿色线程**

查看 README 相关图示

![process](/img/2022-07-07-Rust-async/process_thread_coroutine.png)

![process](/img/2022-07-07-Rust-async/swap_problem.jpg)

可以说，异步任务的行为是模仿 线程 来抽象。

1. 线程在进程内，异步任务在线程内。
2. 线程可被调度切换（Linux默认抢占式），异步任务也可以被调度（协作式而非抢占式）。区别在于，异步任务只在用户态，没有线程的上下文切换开销。
3. 线程有上下文信息，异步任务也有上下文信息。
4. 线程间可以通信，异步任务之间也可以通信。
5. 线程间有竞争，异步任务之间也有竞争。

整个异步编程概念，包括异步语法、异步运行时都是围绕如何建立这种「绿色线程」抽象而成的。

什么是[绿色线程](https://zh.m.wikipedia.org/zh-hans/%E7%BB%BF%E8%89%B2%E7%BA%BF%E7%A8%8B)？

> 在计算机程序设计中，**绿色线程**是一种由[运行环境](https://zh.m.wikipedia.org/wiki/运行时系统)或[虚拟机](https://zh.m.wikipedia.org/wiki/虛擬機器)调度，而不是由本地底层操作系统调度的线程。绿色线程并不依赖底层的系统功能，模拟实现了多线程的运行，这种线程的管理调配发生在用户空间而不是内核空间，所以它们可以在没有原生线程支持的环境中工作。

## Future和Futures-rs介绍

[Future](https://doc.rust-lang.org/std/future/index.html)：提供了异步计算。

Future trait：

```rust
pub trait Future {
    type Output;

    fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output>;
}
```

其中poll方法很重要，尝试去将future得到一个最终的值。如果最终的值没有准备好，也不会进行阻塞。

[task](https://doc.rust-lang.org/std/task/index.html)：可以理解为创建绿色线程。

其中重要的是Enum [std](https://doc.rust-lang.org/std/index.html)::[task](https://doc.rust-lang.org/std/task/index.html)::[Poll](https://doc.rust-lang.org/std/task/enum.Poll.html#)

```rust
pub enum Poll<T> {
    Ready(T),
    Pending,
}
```

[futures-rs](https://github.com/rust-lang/futures-rs)：对future做了扩展。

## 编写异步echo服务

什么是[文件描述符](https://zh.m.wikipedia.org/zh-hans/%E6%96%87%E4%BB%B6%E6%8F%8F%E8%BF%B0%E7%AC%A6)？

> **文件描述符**（File descriptor）是计算机科学中的一个术语，是一个用于表述指向[文件](https://zh.m.wikipedia.org/wiki/文件)的引用的抽象化概念。
>
> 文件描述符在形式上是一个非负整数。实际上，它是一个索引值，指向[内核](https://zh.m.wikipedia.org/wiki/内核)为每一个[进程](https://zh.m.wikipedia.org/wiki/进程)所维护的该进程打开文件的记录表。当程序打开一个现有文件或者创建一个新文件时，内核向进程返回一个文件描述符。在[程序设计](https://zh.m.wikipedia.org/wiki/程序设计)中，一些涉及底层的程序编写往往会围绕着文件描述符展开。但是文件描述符这一概念往往只适用于[UNIX](https://zh.m.wikipedia.org/wiki/UNIX)、[Linux](https://zh.m.wikipedia.org/wiki/Linux)这样的操作系统。

代码见[async-echo-server](https://github.com/ZhangHanDong/inviting-rust/tree/master/async-echo-server)。

流程图：

![image-20220816104218814](/img/2022-07-07-Rust-async/image-20220816104218814.png)

先看看文件的结构：

* main
* epoll
* tcp_listener
* executor
* reactor
* util

tcp_listener:

![image-20220816104414073](/img/2022-07-07-Rust-async/image-20220816104414073.png)

这里主要是几个异步计算的实现，也就是AccpectFuture、ReadFuture、WriteFuture。

epoll：Linux下epoll的封装。

![image-20220816104808295](/img/2022-07-07-Rust-async/image-20220816104808295.png)

主要就是调用系统调用，重要的是epoll的三个方法epoll_create、epoll_ctl、epoll_wait。

executor：执行器，怎样调度这些绿色线程，如何执行任务。

![image-20220816105016908](/img/2022-07-07-Rust-async/image-20220816105016908.png)

reactor：添加事件，检测到事件后唤醒事件，让执行器去执行。

![image-20220816105146577](/img/2022-07-07-Rust-async/image-20220816105146577.png)

## 深入理解异步Task模型

回顾 Rust 异步 task 模型

```text
    +------------------------------------------------------------------+
    |                                                                  |
    |    +--------------------------------------------------------+    |
    |    |                                                        |    |
    |    |   +-------------------------------------------------+  |    |
    |    |   |-------------+ +----------+ +--------------+     |  |    |
    |    |   || futureobj  | | futureobj| |  futureobj   |     |  |    |
    |    |   +-------------+ +----------+ +--------------+     |  |    |
    |    |   | 协 程  task                                      |  |    |
    |    |   +-------------------------------------------------+  |    |
    |    |                                                        |    |
    |    | 线 程                                                   |    |
    |    +--------------------------------------------------------+    |
    |                                                                  |
    |                                                                  |
    |    +--------------------------------------------------------+    |
    |    |  +--------------------------------------------------+  |    |
    |    |  |                                                  |  |    |
    |    |  |   +------------+ +-------------------------------+  |    |
    |    |  |   | futureobj  | |  futureobj      || futureobj ||  |    |
    |    |  |   +------------+ +-------------------------------+  |    |
    |    |  |  协 程 task                                      |  |    |
    |    |  +--------------------------------------------------+  |    |
    |    | 线 程                                                   |    |
    |    +--------------------------------------------------------+    |
    |                                                                  |
    | 进  程                                                            |
    +------------------------------------------------------------------+
```

什么是[协程](https://zh.m.wikipedia.org/zh-hans/%E5%8D%8F%E7%A8%8B)？

> **协程**（英语：coroutine）是计算机程序的一类组件，推广了[协作式多任务](https://zh.m.wikipedia.org/wiki/协作式多任务)的[子例程](https://zh.m.wikipedia.org/wiki/子例程)，允许执行被挂起与被恢复。相对子例程而言，协程更为一般和灵活，但在实践中使用没有子例程那样广泛。协程更适合于用来实现彼此熟悉的程序组件，如[协作式多任务](https://zh.m.wikipedia.org/wiki/协作式多任务)、[异常处理](https://zh.m.wikipedia.org/wiki/异常处理)、[事件循环](https://zh.m.wikipedia.org/wiki/事件循环)、[迭代器](https://zh.m.wikipedia.org/wiki/迭代器)、[无限列表](https://zh.m.wikipedia.org/wiki/惰性求值)和[管道](https://zh.m.wikipedia.org/wiki/管道_(软件))。

同线程的比较

> 协程非常类似于[线程](https://zh.m.wikipedia.org/wiki/线程)。但是协程是[协作式](https://zh.m.wikipedia.org/wiki/协作式多任务)多任务的，而线程典型是[抢占式](https://zh.m.wikipedia.org/wiki/抢占式多任务)[多任务](https://zh.m.wikipedia.org/wiki/多任务)的。这意味着协程提供[并发性](https://zh.m.wikipedia.org/wiki/并发性)而非[并行性](https://zh.m.wikipedia.org/wiki/并行计算)。协程超过线程的好处是它们可以用于[硬性实时](https://zh.m.wikipedia.org/wiki/实时计算)的语境（在协程之间的[切换](https://zh.m.wikipedia.org/wiki/上下文切换)不需要涉及任何[系统调用](https://zh.m.wikipedia.org/wiki/系统调用)或任何[阻塞](https://zh.m.wikipedia.org/wiki/阻塞_(计算))调用），这里不需要用来守卫[关键区段](https://zh.m.wikipedia.org/wiki/关键区段)的同步性原语（primitive）比如[互斥锁](https://zh.m.wikipedia.org/wiki/互斥锁)、信号量等，并且不需要来自操作系统的支持。有可能以一种对调用代码透明的方式，使用抢占式调度的线程实现协程，但是会失去某些利益（特别是对硬性实时操作的适合性和相对廉价的相互之间切换）。
>
> [线程](https://zh.m.wikipedia.org/wiki/线程)是[协作式](https://zh.m.wikipedia.org/wiki/协作式多任务)多任务的轻量级线程，本质上描述了同协程一样的概念。其区别，如果一定要说有的话，是协程是语言层级的构造，可看作一种形式的[控制流](https://zh.m.wikipedia.org/wiki/控制流)，而线程是系统层级的构造，可看作恰巧没有并行运行的线程。这两个概念谁有优先权是争议性的：线程可看作为协程的一种实现[[6\]](https://zh.m.wikipedia.org/zh-hans/协程#cite_note-flounder-6)，也可看作实现协程的基底[[7\]](https://zh.m.wikipedia.org/zh-hans/协程#cite_note-msdn-wrap-7)。

1. 理解 leaf-futures vs Non-leaf-futures (async/await)
   * 叶子futures：像代码示例中的AccpectFuture、ReadFuture、WriteFuture，要和Reactor打交道，比较底层。
   * 非叶子futures：用async或者await实现，业务层面的。
2. 理解 Waker：

> 当事件源注册该Future将在某个事件上等待时，它必须存储唤醒程序，以便以后可以调用唤醒来开始唤醒阶段。 为了引入并发性，能够同时等待多个事件非常重要，因此唤醒器不可能由单个事件源唯一拥有。 结果，Waker类型需要是实现 Clone 的。

- Struct [std](https://doc.rust-lang.org/std/index.html)::[task](https://doc.rust-lang.org/std/task/index.html)::[Waker](https://doc.rust-lang.org/std/task/struct.Waker.html#)

  ```rust
  #[repr(transparent)]
  #[stable(feature = "futures_api", since = "1.36.0")]
  pub struct Waker {
      waker: RawWaker,
  }
  ```

  Waker 可以唤醒一个任务，通过通知执行器。

  Waker实现了一些trait。

  ```rust
  #[stable(feature = "futures_api", since = "1.36.0")]
  impl Unpin for Waker {}
  #[stable(feature = "futures_api", since = "1.36.0")]
  unsafe impl Send for Waker {}
  #[stable(feature = "futures_api", since = "1.36.0")]
  unsafe impl Sync for Waker {}
  ```

  ```rust
  #[derive(PartialEq, Debug)]
  #[stable(feature = "futures_api", since = "1.36.0")]
  pub struct RawWaker {
      /// A data pointer, which can be used to store arbitrary data as required
      /// by the executor. This could be e.g. a type-erased pointer to an `Arc`
      /// that is associated with the task.
      /// The value of this field gets passed to all functions that are part of
      /// the vtable as the first parameter.
      data: *const (),
      /// Virtual function pointer table that customizes the behavior of this waker.
      vtable: &'static RawWakerVTable,
  }
  ```

  这里Rawwaker有两个内容，一个data可以保存上下文，一个vtable是虚表。

3. 理解并发（waker 并发 和 poll 并发）

深入 Futures-rs:

- [Future](https://doc.rust-lang.org/std/future/index.html) and [task](https://doc.rust-lang.org/std/task/index.html)
- [futures-rs](https://github.com/rust-lang/futures-rs)
- [futures-lite](https://github.com/smol-rs/futures-lite)

## async-await语法背后

### 历史

处理异步事件的三种方式：

- Callback
- Promise/Future
- async/await

async/await 是目前体验最好的方式，Rust 要支持它并不容易。

### async/await 语法介绍

参考：[Asynchronous Programming in Rust](https://rust-lang.github.io/async-book/01_getting_started/01_chapter.html)

`async` 两种用法：`async fn ` 函数 和 `async {}` 块。

```rust
// async 函数，真正会返回 `Future<Output = u8>`，而不是表面看上去的 `u8`
async fn foo() -> u8 { 5 }

// async 块用法，返回 `impl Future<Output = u8>`
fn bar() -> impl Future<Output = u8> {
    // 这里 `async` 块返回  `impl Future<Output = u8>`
    async {
        let x: u8 = foo().await;
        x + 5
    }
}
```

`await` 将暂停当前函数的执行，直到执行者将 Future 结束为止。这为其他 Future 任务提供了计算的机会。

## 生成器

Future 底层实现依赖于 生成器。 `async/await` 对应底层生成器 `resume/yield` 。

```rust
#![feature(generators, generator_trait)]
use std::ops::Generator;
use std::pin::Pin;

fn main() {
    let mut gen = || {
        yield 1;
        yield 2;
        yield 3;
        return 4;
    };
    // for _ in 0..4 {
    //     // 为了给嵌入式支持异步，多传入了一个空的unit给resume方法
    //     let c = Pin::new(&mut gen).resume(());
    //     println!("{:?}", c);
    // }


    let c = Pin::new(&mut gen).resume(());
    println!("{:?}", c);
    let c = Pin::new(&mut gen).resume(());
    println!("{:?}", c);
    let c = Pin::new(&mut gen).resume(());
    println!("{:?}", c);
    let c = Pin::new(&mut gen).resume(());
    println!("{:?}", c);
}
```

生成等价代码：

```rust
#![allow(unused)]
#![feature(generators, generator_trait)]
use std::ops::{Generator, GeneratorState};
use std::pin::Pin;

enum __Gen {
    // (0) 初始状态
    Start,
    // (1) resume方法执行以后
    State1(State1),
    // (2) resume方法执行以后
    State2(State2),
    // (3) resume方法执行以后
    State3(State3),
    // (4) resume方法执行以后，正好完成
    Done
}


struct State1 { x: u64 }
struct State2 { x: u64 }
struct State3 { x: u64 }

impl Generator for __Gen {
    type Yield = u64;
    type Return = u64;

    fn resume(self: Pin<&mut Self>, _: ()) -> GeneratorState<u64, u64> {
        let mut_ref = self.get_mut();
        match std::mem::replace(mut_ref, __Gen::Done) {
            __Gen::Start => {
                *mut_ref = __Gen::State1(State1{x: 1});
                GeneratorState::Yielded(1)
            }
            __Gen::State1(State1{x: 1}) => {
                *mut_ref  = __Gen::State2(State2{x: 2});
                GeneratorState::Yielded(2)
        }
            __Gen::State2(State2{x: 2}) => {
                *mut_ref = __Gen::State3(State3{x: 3});
                GeneratorState::Yielded(3)
            }
            __Gen::State3(State3{x: 3}) => {
                *mut_ref  = __Gen::Done;
                GeneratorState::Complete(4)
            }
            _ => {
                panic!("generator resumed after completion")
            }
        }
    }
}

fn main(){
    let mut gen = {
        __Gen::Start
    };

    for _ in 0..4 {
        println!("{:?}", unsafe{ Pin::new(&mut gen).resume(())});
    }
}
```

生成器基本用法：

```rust
#![allow(unused)]
#![feature(generators, generator_trait)]
use std::pin::Pin;

use std::ops::Generator;

pub fn up_to(limit: u64) -> impl Generator<Yield = u64, Return = u64> {
    move || {
        for x in 0..limit {
            yield x;
        }
        return limit;
    }
}
fn main(){
    let a = 10;
    let mut b = up_to(a);
    unsafe {
        for _ in 0..=10{
            let c = Pin::new(&mut b).resume(());
            println!("{:?}", c);
        }
    }
}
```

生成器变身为迭代器：

```rust
#![allow(unused)]
#![feature(generators, generator_trait)]
use std::pin::Pin;

use std::ops::{Generator, GeneratorState};

pub fn up_to() -> impl Generator<Yield = u64, Return = ()> {
    move || {
        let mut x = 0;
        loop {
            x += 1;
            yield x;
        }
        return ();
    }
}
fn main(){
    let mut gen = up_to();
    unsafe {
    for _ in 0..10{
        match Pin::new(&mut gen).resume(()) {
            GeneratorState::Yielded(i) => println!("{:?}", i),
            _ => println!("Completed"),
        }
    }
    }
}
```

生成器变身为 Future:

```rust
#![allow(unused)]
#![feature(generators, generator_trait)]

use std::ops::{Generator, GeneratorState};
use std::pin::Pin;

pub fn up_to(limit: u64) -> impl Generator<Yield = (), Return = Result<u64, ()>> {
    move || {
        for x in 0..limit {
            yield ();
        }
        return Ok(limit);
    }
}
fn main(){
    let limit = 2;
    let mut gen = up_to(limit);
    unsafe {
    for i in 0..=limit{
        match Pin::new(&mut gen).resume(()) {
            GeneratorState::Yielded(v) => println!("resume {:?} : Pending", i),
            GeneratorState::Complete(v) => println!("resume {:?} : Ready", i),
        }
    }
    }
}
```

在标准库Future内部，可以从Generator转换为Future。

```rust
#[lang = "from_generator"]
#[doc(hidden)]
#[unstable(feature = "gen_future", issue = "50547")]
#[rustc_const_unstable(feature = "gen_future", issue = "50547")]
#[inline]
pub const fn from_generator<T>(gen: T) -> impl Future<Output = T::Return>
where
    T: Generator<ResumeTy, Yield = ()>,
{
    #[rustc_diagnostic_item = "gen_future"]
    struct GenFuture<T: Generator<ResumeTy, Yield = ()>>(T);

    // We rely on the fact that async/await futures are immovable in order to create
    // self-referential borrows in the underlying generator.
    impl<T: Generator<ResumeTy, Yield = ()>> !Unpin for GenFuture<T> {}

    impl<T: Generator<ResumeTy, Yield = ()>> Future for GenFuture<T> {
        type Output = T::Return;
        fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output> {
            // SAFETY: Safe because we're !Unpin + !Drop, and this is just a field projection.
            let gen = unsafe { Pin::map_unchecked_mut(self, |s| &mut s.0) };

            // Resume the generator, turning the `&mut Context` into a `NonNull` raw pointer. The
            // `.await` lowering will safely cast that back to a `&mut Context`.
            match gen.resume(ResumeTy(NonNull::from(cx).cast::<Context<'static>>())) {
                GeneratorState::Yielded(()) => Poll::Pending,
                GeneratorState::Complete(x) => Poll::Ready(x),
            }
        }
    }

    GenFuture(gen)
}
```

## no-std异步生态介绍

* [futures-micro](https://github.com/irrustible/futures-micro)  对futures的最小的实现
* [embassy](https://github.com/embassy-rs/embassy) 嵌入式生态
* [executor for no-std](https://github.com/richardanaya/executor) no-std的执行器，可以用在嵌入式及WebAssembly上

## 一个异步缓存代码实现

代码见[retainer](https://github.com/ChaosStudyGroup/retainer)。

下面分析下代码结构，首先是 entry.rs。

缓存的entry结构：

```rust
#[derive(Debug)]
pub struct CacheEntry<V> {
    pub(crate) value: V,
    pub(crate) expiration: Option<CacheExpiration>,
}
```

有相应的值value以及过期时间expiration，这里过期时间是一个Option，因为有的可以没有过期时间。

CacheExpiration：

```rust
#[derive(Debug)]
pub struct CacheExpiration {
    instant: Instant,
}
```

代表缓存中的过期时间，这里对Instant有一个包装，方便实现一些方法。

```rust
#[derive(Debug)]
pub struct CacheEntryReadGuard<'a, V> {
    pub(crate) entry: *const CacheEntry<V>,
    pub(crate) marker: PhantomData<&'a CacheEntry<V>>,
}
```

对底层缓存结构的引用的一个读保护。当使用锁的时候，这个结构可以返回一个指向内部缓存 entries 的引用，也就是一个指针。同时CacheEntryReadGuard实现了Send和Sync。

```rust
// Stores a raw pointer to `T`, so if `T` is `Sync`, the lock guard over `T` is `Send`.
unsafe impl<V> Send for CacheEntryReadGuard<'_, V> where V: Sized + Sync {}
unsafe impl<V> Sync for CacheEntryReadGuard<'_, V> where V: Sized + Send + Sync {}
```

cache.rs

缓存结构：

```rust
pub struct Cache<K, V> {
    store: RwLock<BTreeMap<K, CacheEntry<V>>>,
    label: String,
}
```

监控是否有过期缓存：

```rust
pub async fn monitor(&self, sample: usize, threshold: f64, frequency: Duration) {
  let mut interval = Interval::platform_new(frequency);
  loop {
    interval.as_mut().await;
    self.purge(sample, threshold).await;
  }
}
```

可以看到 purge 函数最终实现异步清理。

这里说下Redis的缓存过期处理方式：

* 惰性删除。当查找一个key，过期时删除。
* 定期删除。每隔一段时间，随机取一定数量的key，看看缓存中是否存在，如果到达一定的阈值比如20%已经过期了，则进行删除，如果没有到达该阈值，则不删除。

关于Redis缓存，参见[Redis缓存总结：淘汰机制、缓存雪崩、数据不一致....](https://segmentfault.com/a/1190000039020626)。

## 异步运行时生态

* tokio 和rust异步标准有些差异，成熟
* async-std 让开发者顺利编写异步，对标准库的异步
* smol 轻量级，比较通用
* glommio 生产级应用
* bastion 高可用分布式容错框架

## smol运行时

先看看smol中的Cargo.toml。

```toml
[dependencies]
async-channel = "1.4.2"
async-executor = "1.3.0"
async-fs = "1.3.0"
async-io = "1.1.2"
async-lock = "2.3.0"
async-net = "1.4.3"
async-process = "1.0.0"
blocking = "1.0.0"
futures-lite = "1.11.0"
once_cell = "1.4.1"
```

smol对这些组件做了一个整合，这些组件也可以单独使用。

### [smol文档](https://docs.rs/smol/latest/smol/)

[Modules](https://docs.rs/smol/latest/smol/#modules)

| Name                                                         | Description                                                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [channel](https://docs.rs/smol/latest/smol/channel/index.html) | An async multi-producer multi-consumer channel.              |
| [fs](https://docs.rs/smol/latest/smol/fs/index.html)         | Async filesystem primitives.                                 |
| [future](https://docs.rs/smol/latest/smol/future/index.html) | Combinators for the [`Future`](https://docs.rs/smol/latest/smol/prelude/trait.Future.html) trait. |
| [io](https://docs.rs/smol/latest/smol/io/index.html)         | Tools and combinators for I/O.                               |
| [lock](https://docs.rs/smol/latest/smol/lock/index.html)     | Async synchronization primitives.                            |
| [net](https://docs.rs/smol/latest/smol/net/index.html)       | Async networking primitives for TCP/UDP/Unix communication.  |
| [prelude](https://docs.rs/smol/latest/smol/prelude/index.html) | Traits [`Future`](https://docs.rs/smol/latest/smol/prelude/trait.Future.html), [`Stream`](https://docs.rs/smol/latest/smol/stream/trait.Stream.html), [`AsyncRead`](https://docs.rs/smol/latest/smol/prelude/trait.AsyncRead.html), [`AsyncWrite`](https://docs.rs/smol/latest/smol/prelude/trait.AsyncWrite.html), [`AsyncBufRead`](https://docs.rs/smol/latest/smol/prelude/trait.AsyncBufRead.html), [`AsyncSeek`](https://docs.rs/smol/latest/smol/prelude/trait.AsyncSeek.html), and their extensions. |
| [process](https://docs.rs/smol/latest/smol/process/index.html) | Async interface for working with processes.                  |
| [stream](https://docs.rs/smol/latest/smol/stream/index.html) | Combinators for the [`Stream`](https://docs.rs/smol/latest/smol/stream/trait.Stream.html) trait. |

[Structs](https://docs.rs/smol/latest/smol/#structs)

| Name                                                         | Description                                                  |
| :----------------------------------------------------------- | ------------------------------------------------------------ |
| [Async](https://docs.rs/smol/latest/smol/struct.Async.html)  | Async adapter for I/O types. 异步适配器，可以将阻塞IO变为非阻塞IO。 |
| [Executor](https://docs.rs/smol/latest/smol/struct.Executor.html) | An async executor. 异步执行器。                              |
| [LocalExecutor](https://docs.rs/smol/latest/smol/struct.LocalExecutor.html) | A thread-local executor. 单线程执行器。                      |
| [Task](https://docs.rs/smol/latest/smol/struct.Task.html)    | A spawned task. 任务。                                       |
| [Timer](https://docs.rs/smol/latest/smol/struct.Timer.html)  | A future that expires at a point in time.                    |
| [Unblock](https://docs.rs/smol/latest/smol/struct.Unblock.html) | Runs blocking I/O on a thread pool.                          |

### smol源码

src/spawn.rs

```rust
use once_cell::sync::Lazy;

/// 将异步任务加入到 task 中，交给 Global 执行器
pub fn spawn<T: Send + 'static>(future: impl Future<Output = T> + Send + 'static) -> Task<T> {
    static GLOBAL: Lazy<Executor<'_>> = Lazy::new(|| {
        let num_threads = {
            // Parse SMOL_THREADS or default to 1.
            std::env::var("SMOL_THREADS")
                .ok()
                .and_then(|s| s.parse().ok())
                .unwrap_or(1)
        };

        for n in 1..=num_threads {
            thread::Builder::new()
                .name(format!("smol-{}", n))
                .spawn(|| loop {
                    // catch_unwind 方法
                    // 可以捕获栈展开，运行panic线程存活并继续运行。使用标准库中的std::panic::catch_unwind()函数。
                    catch_unwind(|| block_on(GLOBAL.run(future::pending::<()>()))).ok();
                })
                .expect("cannot spawn executor thread");
        }

        Executor::new()
    });

    GLOBAL.spawn(future)
}
```



### 聊天框例子
#### chat-server
聊天框服务器的事件：

```rust
/// An event on the chat server.
enum Event {
    /// A client has joined.
    Join(SocketAddr, Arc<Async<TcpStream>>),

    /// A client has left.
    Leave(SocketAddr),

    /// A client sent a message.
    Message(SocketAddr, String),
}
```

向客户端分发事件：

```rust
/// Dispatches events to clients.
async fn dispatch(receiver: Receiver<Event>) -> io::Result<()> {
    // Currently active clients.
    let mut map = HashMap::<SocketAddr, Arc<Async<TcpStream>>>::new();

    // Receive incoming events.
    while let Ok(event) = receiver.recv().await {
        // Process the event and format a message to send to clients.
        let output = match event {
            Event::Join(addr, stream) => {
                map.insert(addr, stream);
                format!("{} has joined\n", addr)
            }
            Event::Leave(addr) => {
                map.remove(&addr);
                format!("{} has left\n", addr)
            }
            Event::Message(addr, msg) => format!("{} says: {}\n", addr, msg),
        };

        // Display the event in the server process.
        print!("{}", output);

        // Send the event to all active clients.
        for stream in map.values_mut() {
            // Ignore errors because the client might disconnect at any point.
            stream.write_all(output.as_bytes()).await.ok();
        }
    }
    Ok(())
}
```

服务端 main 函数：
```rust
fn main() -> io::Result<()> {
    smol::block_on(async {
        // Create a listener for incoming client connections.
        let listener = Async::<TcpListener>::bind(([127, 0, 0, 1], 6000))?;

        // Intro messages.
        println!("Listening on {}", listener.get_ref().local_addr()?);
        println!("Start a chat client now!\n");

        // Spawn a background task that dispatches events to clients.
        // 通过 channel 实现消息传递
        let (sender, receiver) = bounded(100);
        // detach 函数
        // 分离任务，让它在后台运行
        smol::spawn(dispatch(receiver)).detach();

        loop {
            // Accept the next connection.
            let (stream, addr) = listener.accept().await?;
            let client = Arc::new(stream);
            let sender = sender.clone();

            // Spawn a background task reading messages from the client.
            smol::spawn(async move {
                // Client starts with a `Join` event.
                sender.send(Event::Join(addr, client.clone())).await.ok();

                // Read messages from the client and ignore I/O errors when the client quits.
                read_messages(sender.clone(), client).await.ok();

                // Client ends with a `Leave` event.
                sender.send(Event::Leave(addr)).await.ok();
            })
            .detach();
        }
    })
}
```

#### chat-client
```rust
fn main() -> io::Result<()> {
    smol::block_on(async {
        // Connect to the server and create async stdin and stdout.
        let stream = Async::<TcpStream>::connect(([127, 0, 0, 1], 6000)).await?;
        let stdin = Unblock::new(std::io::stdin());
        let mut stdout = Unblock::new(std::io::stdout());

        // Intro messages.
        println!("Connected to {}", stream.get_ref().peer_addr()?);
        println!("My nickname: {}", stream.get_ref().local_addr()?);
        println!("Type a message and hit enter!\n");

        let reader = &stream;
        let mut writer = &stream;

        // Wait until the standard input is closed or the connection is closed.
        future::race(
            async {
                let res = io::copy(stdin, &mut writer).await;
                println!("Quit!");
                res
            },
            async {
                let res = io::copy(reader, &mut stdout).await;
                println!("Server disconnected!");
                res
            },
        )
        .await?;

        Ok(())
    })
}
```

### futures_lite
文档：[futures_lite](https://docs.rs/futures-lite/latest/futures_lite/)，futures库的轻量版。

[Modules](https://docs.rs/futures-lite/latest/futures_lite/#modules)

| Name                                                         | Description                                                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [future](https://docs.rs/futures-lite/latest/futures_lite/future/index.html) | Combinators for the [`Future`](https://doc.rust-lang.org/nightly/core/future/future/trait.Future.html) trait. |
| [io](https://docs.rs/futures-lite/latest/futures_lite/io/index.html) | Tools and combinators for I/O.                               |
| [prelude](https://docs.rs/futures-lite/latest/futures_lite/prelude/index.html) | Traits [`Future`](https://doc.rust-lang.org/nightly/core/future/future/trait.Future.html), [`Stream`](https://docs.rs/futures-core/0.3.15/futures_core/stream/trait.Stream.html), [`AsyncRead`](https://docs.rs/futures-io/0.3.15/futures_io/if_std/trait.AsyncRead.html), [`AsyncWrite`](https://docs.rs/futures-io/0.3.15/futures_io/if_std/trait.AsyncWrite.html), [`AsyncBufRead`](https://docs.rs/futures-io/0.3.15/futures_io/if_std/trait.AsyncBufRead.html), [`AsyncSeek`](https://docs.rs/futures-io/0.3.15/futures_io/if_std/trait.AsyncSeek.html), and their extensions. |
| [stream](https://docs.rs/futures-lite/latest/futures_lite/stream/index.html) | Combinators for the [`Stream`](https://docs.rs/futures-core/0.3.15/futures_core/stream/trait.Stream.html) trait. |

主要就是上述四个模块。

### async_io
文档：[async_io](https://docs.rs/async-io/latest/async_io/)

在async_io/lib.rs中定义了两个结构体，一个是Timer，另一个是Async。
* Timer: 在某个时间点到期的 Future。
* Async: 在异步编程中在标准网络类型（还有许多其他类型的）中使用的一个适配器。

```rust
#[derive(Debug)]
pub struct Timer {
    /// This timer's ID and last waker that polled it.
    ///
    /// When this field is set to `None`, this timer is not registered in the reactor.
    id_and_waker: Option<(usize, Waker)>,

    /// The next instant at which this timer fires.
    when: Instant,

    /// The period.
    period: Duration,
}

#[derive(Debug)]
pub struct Async<T> {
    /// A source registered in the reactor.
    source: Arc<Source>,

    /// The inner I/O handle.
    io: Option<T>,
}
```

这里用到了[polling](https://docs.rs/polling/latest/polling/) crate。这个库实现了跨平台的poll，提供了很多接口。

> Portable interface to epoll, kqueue, event ports, and wepoll.
> Supported platforms:
>
> * epoll : Linux, Android
> * kqueue : macOS, iOS, FreeBSD, NetBSD, OpenBSD, DragonFly BSD
> * event ports : illumos, Solaris
> * poll : VxWorks, Fuchsia, other Unix systems
> * wepoll : Windows

下面看看 async_io/reactor.rs

```rust
pub(crate) struct Reactor {
    /// Portable bindings to epoll/kqueue/event ports/wepoll.
    ///
    /// This is where I/O is polled, producing I/O events.
    poller: Poller,

    /// Ticker bumped before polling.
    ///
    /// This is useful for checking what is the current "round" of `ReactorLock::react()` when
    /// synchronizing things in `Source::readable()` and `Source::writable()`. Both of those
    /// methods must make sure they don't receive stale I/O events - they only accept events from a
    /// fresh "round" of `ReactorLock::react()`.
    ticker: AtomicUsize,

    /// Registered sources.
    sources: Mutex<Slab<Arc<Source>>>,

    /// Temporary storage for I/O events when polling the reactor.
    ///
    /// Holding a lock on this event list implies the exclusive right to poll I/O.
    events: Mutex<Vec<Event>>,

    /// An ordered map of registered timers.
    ///
    /// Timers are in the order in which they fire. The `usize` in this type is a timer ID used to
    /// distinguish timers that fire at the same time. The `Waker` represents the task awaiting the
    /// timer.
    timers: Mutex<BTreeMap<(Instant, usize), Waker>>,

    /// A queue of timer operations (insert and remove).
    ///
    /// When inserting or removing a timer, we don't process it immediately - we just push it into
    /// this queue. Timers actually get processed when the queue fills up or the reactor is polled.
    timer_ops: ConcurrentQueue<TimerOp>,
}
```

还有 async_io/driver.rs，其中有main_loop函数，对async_io线程进行主要的循环，另外还有block_on函数，在有异步任务时阻塞当前的线程，在空闲的时候处理IO事件。

### async_task
文档： [async_task](https://docs.rs/async-task/latest/async_task/)


Task abstraction for building executors.

To spawn a future onto an executor, we first need to allocate it on the heap and keep some state attached to it. The state indicates whether the future is ready for polling, waiting to be woken up, or completed. Such a stateful future is called a task.

All executors have a queue that holds scheduled tasks:
```rust
let (sender, receiver) = flume::unbounded();
```
这里用到了第三方库[flume](https://docs.rs/flume/0.10.14/flume/)。它比 crossbeam-channel 更加轻量，实现了mpmc。

接着来看 async_task 的函数：
* spawn	创建一个新任务。
* spawn_local 创建一个新的本地任务。
* spawn_unchecked⚠	Creates a new task without Send, Sync, and 'static bounds.

async_task 的结构：
* FallibleTask:	A spawned task with a fallible response.
* Runnable:	A handle to a runnable task.
* Task:	A spawned task.

在 async_task/raw.rs 中定义了底层的TaskVTable、TaskLayout、RawTask。

在 async_task/header.rs 中定义了结构体Header。
```rust
pub(crate) struct Header {
    /// Current state of the task.
    ///
    /// Contains flags representing the current state and the reference count.
    pub(crate) state: AtomicUsize,

    /// The task that is blocked on the `Task` handle.
    ///
    /// This waker needs to be woken up once the task completes or is closed.
    pub(crate) awaiter: UnsafeCell<Option<Waker>>,

    /// The virtual table.
    ///
    /// In addition to the actual waker virtual table, it also contains pointers to several other
    /// methods necessary for bookkeeping the heap-allocated task.
    pub(crate) vtable: &'static TaskVTable,
}
```
相当于做一些任务的处理，提供了notify、take、register三个函数。

这里面任务的状态state是在state.rs中定义的。
```rust
/// Set if the task is scheduled for running.
pub(crate) const SCHEDULED: usize = 1 << 0;

/// Set if the task is running.
pub(crate) const RUNNING: usize = 1 << 1;

/// Set if the task has been completed.
pub(crate) const COMPLETED: usize = 1 << 2;

/// Set if the task is closed.
pub(crate) const CLOSED: usize = 1 << 3;

/// Set if the `Task` still exists.
pub(crate) const TASK: usize = 1 << 4;

/// Set if the `Task` is awaiting the output.
pub(crate) const AWAITER: usize = 1 << 5;

/// Set if an awaiter is being registered.
pub(crate) const REGISTERING: usize = 1 << 6;

/// Set if the awaiter is being notified.
pub(crate) const NOTIFYING: usize = 1 << 7;

/// A single reference.
pub(crate) const REFERENCE: usize = 1 << 8;
```

### async_executor
异步执行器。
```rust
use async_executor::Executor;
use futures_lite::future;

// Create a new executor.
let ex = Executor::new();

// Spawn a task.
let task = ex.spawn(async {
    println!("Hello world");
});

// Run the executor until the task completes.
future::block_on(ex.run(task));
```

### blocking
文档：[blocking](https://docs.rs/blocking/latest/blocking/)

在异步程序里提供了一个线程池用于隔离阻塞的IO。

unblock 函数：对阻塞的代码进行异步化。

Unblock 类型：对阻塞的IO进行异步化。通过维护一个环形队列来进行处理。

[async-fs](https://docs.rs/async-fs/latest/async_fs/) 其中用到了 blocking 这个库。

## async-std
文档：[async_std](https://docs.rs/async-std/1.12.0/async_std/)，rust标准库的异步版本。

task的使用以及抽象出来的API接口类似线程，易于上手。

async_std 和 smol 运行时一些架构以及组件都是公用的，并且它提供了一个类似于线程的一个异步 task 的抽象，并且它还兼容 tokio。

## tokio
github仓库地址：[tokio](https://github.com/tokio-rs/tokio)

官方网站：[tokio.rs](https://tokio.rs/)

文档：[tokio](https://docs.rs/tokio/latest/tokio/)

官方文档介绍：

A runtime for writing reliable network applications without compromising speed.

Tokio is an event-driven, non-blocking I/O platform for writing asynchronous applications with the Rust programming language. At a high level, it provides a few major components:
* Tools for working with asynchronous tasks, including synchronization primitives and channels and timeouts, sleeps, and intervals.
* APIs for performing asynchronous I/O, including TCP and UDP sockets, filesystem operations, and process and signal management.
* A runtime for executing asynchronous code, including a task scheduler, an I/O driver backed by the operating system’s event queue (epoll, kqueue, IOCP, etc…), and a high performance timer.

Remark: loom 模块用来检测异步并发，也可以检测同步并发。

### mini-redis
* github仓库地址：[mini-redis](https://github.com/tokio-rs/mini-redis)

[Redis教程](https://www.runoob.com/redis/redis-tutorial.html)

`#[tokio::main]`宏，定义在tokio-macros/lib.rs中。
```rust
#[tokio::main]
async fn main() {
    println!("Hello world");
}
```
宏展开之后为：
```rust
fn main() {
    tokio::runtime::Builder::new_multi_thread()
        .enable_all()
        .build()
        .unwrap()
        .block_on(async {
            println!("Hello world");
        })
}
```

```rust
#[proc_macro_attribute]
#[cfg(not(test))] // Work around for rust-lang/rust#62127
pub fn main(args: TokenStream, item: TokenStream) -> TokenStream {
    entry::main(args, item, true)
}
```
在entry.rs中：
```rust
#[cfg(not(test))] // Work around for rust-lang/rust#62127
pub(crate) fn main(args: TokenStream, item: TokenStream, rt_multi_thread: bool) -> TokenStream {
    // If any of the steps for this macro fail, we still want to expand to an item that is as close
    // to the expected output as possible. This helps out IDEs such that completions and other
    // related features keep working.
    let input: syn::ItemFn = match syn::parse(item.clone()) {
        Ok(it) => it,
        Err(e) => return token_stream_with_error(item, e),
    };

    let config = if input.sig.ident == "main" && !input.sig.inputs.is_empty() {
        let msg = "the main function cannot accept arguments";
        Err(syn::Error::new_spanned(&input.sig.ident, msg))
    } else {
        AttributeArgs::parse_terminated
            .parse(args)
            .and_then(|args| build_config(input.clone(), args, false, rt_multi_thread))
    };

    match config {
        Ok(config) => parse_knobs(input, false, config),
        Err(e) => token_stream_with_error(parse_knobs(input, false, DEFAULT_ERROR_CONFIG), e),
    }
}
```

还有一个select宏，官方教程参见[Select](https://tokio.rs/tokio/tutorial/select)。例如：
```rust
use tokio::sync::oneshot;

#[tokio::main]
async fn main() {
    let (tx1, rx1) = oneshot::channel();
    let (tx2, rx2) = oneshot::channel();

    tokio::spawn(async {
        let _ = tx1.send("one");
    });

    tokio::spawn(async {
        let _ = tx2.send("two");
    });

    tokio::select! {
        val = rx1 => {
            println!("rx1 completed first with {:?}", val);
        }
        val = rx2 => {
            println!("rx2 completed first with {:?}", val);
        }
    }
}
```

tokio的运行时是通过Builder来创建的。tokio中分两种线程，core线程专门处理异步，blocking线程处理同步。

```rust
pub struct Builder {
    /// Runtime type
    kind: Kind,

    /// Whether or not to enable the I/O driver
    enable_io: bool,

    /// Whether or not to enable the time driver
    enable_time: bool,

    /// Whether or not the clock should start paused.
    start_paused: bool,

    /// The number of worker threads, used by Runtime.
    ///
    /// Only used when not using the current-thread executor.
    worker_threads: Option<usize>, // core 线程

    /// Cap on thread usage.
    max_blocking_threads: usize, // blocking线程最大数量

    /// Name fn used for threads spawned by the runtime.
    pub(super) thread_name: ThreadNameFn,

    /// Stack size used for threads spawned by the runtime.
    pub(super) thread_stack_size: Option<usize>,

    /// Callback to run after each thread starts.
    pub(super) after_start: Option<Callback>,

    /// To run before each worker thread stops
    pub(super) before_stop: Option<Callback>,

    /// To run before each worker thread is parked.
    pub(super) before_park: Option<Callback>,

    /// To run after each thread is unparked.
    pub(super) after_unpark: Option<Callback>,

    /// Customizable keep alive timeout for BlockingPool
    pub(super) keep_alive: Option<Duration>,

    /// How many ticks before pulling a task from the global/remote queue?
    pub(super) global_queue_interval: u32,

    /// How many ticks before yielding to the driver for timer and I/O events?
    pub(super) event_interval: u32,

    #[cfg(tokio_unstable)]
    pub(super) unhandled_panic: UnhandledPanic,
}
```

tokio/park：

有事件唤醒线程让它去spawn task，如果没有的话就等待 park。

tokio中的任务队列tokio/src/runtime/thread_pool/worker.rs。

```rust
/// A scheduler worker
pub(super) struct Worker {
    /// Reference to shared state
    shared: Arc<Shared>,

    /// Index holding this worker's remote state
    index: usize,

    /// Used to hand-off a worker's core to another thread.
    core: AtomicCell<Core>,
}

/// Core data
struct Core {
    /// Used to schedule bookkeeping tasks every so often.
    tick: u32,

    /// When a task is scheduled from a worker, it is stored in this slot. The
    /// worker will check this slot for a task **before** checking the run
    /// queue. This effectively results in the **last** scheduled task to be run
    /// next (LIFO). This is an optimization for message passing patterns and
    /// helps to reduce latency.
    lifo_slot: Option<Notified>, // 后进先出队列，做了优化

    /// The worker-local run queue.
    run_queue: queue::Local<Arc<Shared>>, // 本地队列

    /// True if the worker is currently searching for more work. Searching
    /// involves attempting to steal from other workers.
    is_searching: bool,

    /// True if the scheduler is being shutdown
    is_shutdown: bool,

    /// Parker
    ///
    /// Stored in an `Option` as the parker is added / removed to make the
    /// borrow checker happy.
    park: Option<Parker>,

    /// Batching metrics so they can be submitted to RuntimeMetrics.
    metrics: MetricsBatch,

    /// Fast random number generator.
    rand: FastRand,

    /// How many ticks before pulling a task from the global/remote queue?
    global_queue_interval: u32,

    /// How many ticks before yielding to the driver for timer and I/O events?
    event_interval: u32,
}
```
提供了一个方法block_in_place，可以将当前线程变为阻塞线程。

```rust
pub(crate) fn block_in_place<F, R>(f: F) -> R
where
    F: FnOnce() -> R,
{
    ...
}
```

### 流处理
文档参考：[tokio_util::codec](https://docs.rs/tokio-util/0.7.3/tokio_util/codec/index.html)。

Adaptors from AsyncRead/AsyncWrite to Stream/Sink

Raw I/O objects work with byte sequences, but higher-level code usually wants to batch these into meaningful chunks, called “frames”.

This module contains adapters to go from streams of bytes, AsyncRead and AsyncWrite, to framed streams implementing Sink and Stream. Framed streams are also known as transports.

tokio-stream抽象了一种帧结果，也提供相应的适配器进行异步读写。如果想更好的进行异步读写，可以使用抽象的codec，也可以使用更底层的Sink/Stream进行处理。

[tonic](https://github.com/hyperium/tonic)：实现gRPC客户端和服务端，自己实现Stream，从而来进行异步读写。

[wrap](https://github.com/seanmonstar/warp)：实现文件上传，也是通过自己实现Stream。

[redis-rs](https://github.com/redis-rs/redis-rs)：redis library。这里用到了解析器组合子[combine](https://github.com/Marwes/combine)。

### 扩展资料
* [Tokio Internals - 源码解析和设计分析](https://tony612.github.io/tokio-internals/)

## 参考资料

[张汉东的Rust实战课](https://time.geekbang.org/course/intro/348)

[张汉东的Rust实战课视频课程代码示例](https://github.com/ZhangHanDong/inviting-rust)

