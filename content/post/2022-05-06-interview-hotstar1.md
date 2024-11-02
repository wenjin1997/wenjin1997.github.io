---
layout:     post

title:      "Hotstar一面"
# subtitle:   "Java并发编程的艺术笔记"
# excerpt: "Java并发编程的艺术笔记"
author:     "谢文进"
date:       2022-05-06
description: "总结Hotstar一面。"
# image: "/img/2022-03-28-The-Art-of-Java-Concurrency-Programming/background.jpg"
published: 2022-05-06 
tags:
     - 面试
     - Java

categories: [ Tech ]
URL: "/2022/05/06/interview-hotstar1/"
---
面试问题：

1. 自我介绍
2. 数学专业为什么转计算机？
3. 本科专业是应用数学，为什么有计算机相关课程。
4. 用过MySQL吗？熟悉事务吗？
这个我没答上来，不太会。
5. 对SpringBoot了解吗？
呜呜呜，还没学到那里。
6. 讲下HashMap。
7. 对计算机网络的了解。
8. TCP和UDP的区别。

下面就是发个链接开始写代码，题目是[209.长度最小的子数组](https://leetcode-cn.com/problems/minimum-size-subarray-sum/)。

我开始想到子数组求和，就要用到前缀和数组，然后用双指针来遍历，但是双指针遍历逻辑有问题。面试官提醒了用滑动窗口，讲了下实现的逻辑，然后让我实现，实现的时候还出错了，报下标越界错误，是判断位置加错了。最后加了一个特判，就结束了。写完后问了复杂度。

写代码这块大概花了一个多小时，是我太菜了。下面就是反问环节。

我问了在编程中会用到哪些数学知识吗？面试官说这边核心代码基本开发完了，如果后面想要深入了解也是可以的，对代码理解还是有帮助的。

问了对Hotstar的了解，哈哈，我了解的不多。面试官就介绍了下。

面试体验超棒，面试官问问题感觉很温柔，算法题也会引导。

呜呜呜，收到了感谢信:(

革命尚未成功，同志仍需努力！