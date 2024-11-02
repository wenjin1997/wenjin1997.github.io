---
layout:     post
title:      "Dan Boneh Cryptography I Week1"
# subtitle:   ""
# excerpt: "Zero Knowledge Proofs:An illustrated primer"
description: "Dan Boneh Cryptography I 课程第一周笔记。"
date:       2022-11-21
author:     "谢文进"
image: "/img/2022-11-21-Dan-Boneh-Cryptography-Week1/background.jpg"
published: 2022-11-21
tags:
    - ZKP
    - 笔记 
    - Dan Boneh Cryptography I

categories: [ ZKP ]
URL: "/2022/11/21/Dan-Boneh-Cryptography-Week1/"
---

开始学习密码学，这是一套不错的课程。官方课程链接[Online Cryptography Course](https://crypto.stanford.edu/~dabo/courses/OnlineCrypto/)。教材[A Graduate Course in Applied Cryptography](http://toc.cryptobook.us/)，Coursera上的链接为[密码学 I](https://www.coursera.org/learn/crypto)。

下面记录第一周的学习笔记。

## **Course Overview**

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled.png)

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%201.png)

密码学无处不在，DVD使用的是CSS加密，Blu-ray使用的是AACS加密。

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%202.png)

常用的HTTPS底层用的是SSL/TLS。安全的交流中间是没有偷听和篡改的。

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%203.png)

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%204.png)

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%205.png)

对称加密系统使用的是同一个key。加密算法是公开的，永远不要使用一个专有的加密算法。

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%206.png)

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%207.png)

密码学不是所有安全问题的解决方案，例如软件漏洞或者工程上的攻击。

## **What is cryptography?**

### Crypto core

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%208.png)

密码学的核心建立密钥，保证交流的可靠和完整性。

### But crypto can do much more

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%209.png)

密码学可以应用在匿名交流中，上图中的匿名是相互的，Alice不知道对方是谁，Bob也不知道对方是谁。

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%2010.png)

密码学应用在匿名电子货币，类似于我们去商店消费，不想让商店知道我们的身份，同时在网络中，要保证电子货币不能重复消费。

### **Protocols**

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%2011.png)

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%2012.png)

在选举或者私有拍卖的例子中，有受信的第三方来进行公布结果。这里有一个重要的理论，任何可以使用受信第三方完成的事也可以不用第三方就能完成。

### **Crypto magic**

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%2013.png)

### **A rigorous science**

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%2014.png)

密码学是一门严谨的学科，要遵循以上三个步骤。

## **History**

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%2015.png)

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%2016.png)

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%2017.png)

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%2018.png)

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%2019.png)

替换式加密的密钥空间很大，是26个字母的全排列。

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%2020.png)

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%2021.png)

破解替换式加密的方法是利用了英语字母出现的频率，字母e出现的频率最高，那么我们可以对截获到的密文统计出现的频率，出现频率最高的字母就对应于字母e。接着再利用二合字母出现的频率。

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%2022.png)

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%2023.png)

当我们知道key中加密字母的长度时，就很好破解。

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%2024.png)

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%2025.png)

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%2026.png)

## **Discrete Probability**

这部分详细介绍见[https://en.wikibooks.org/wiki/High_School_Mathematics_Extensions/Discrete_Probability](https://en.wikibooks.org/wiki/High_School_Mathematics_Extensions/Discrete_Probability)。

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%2027.png)

### Events

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%2028.png)

### The union bound

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%2029.png)

### Random Variables

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%2030.png)

### The uniform random variable

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%2031.png)

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%2032.png)

### Randomized algorithms

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%2033.png)

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%2034.png)

### Independence

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%2035.png)

### XOR

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%2036.png)

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%2037.png)

异或中重要的一个定理是，Y是一个随机变量，X是一个独立的均匀分布变量，Y与X异或之后是一个均匀分布变量。

### **The birthday paradox**

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%2038.png)

当$n=1.2 \times \sqrt{|U|}$时，$U$ 中存在两个变量相等的概率大于等于$1/2$。常识认为可能是$|U|/2$，因此也叫做悖论。

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%2039.png)

## The One Time Pad

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%2040.png)

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%2041.png)

key和要加密的消息的长度一样长。

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%2042.png)

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%2043.png)

key也可以算出来，是m和c的异或。

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%2044.png)

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%2045.png)

什么是安全的加密呢？香农的定义是，不能从密文中得到任何关于原文的信息。

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%2046.png)

用概率来定义 perfect secrecy，也就是从密文中无法区分任意两个原文，得到两个不同message的概率是相同的。

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%2047.png)

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%2048.png)

OTP是有perfect安全性的。

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%2049.png)

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%2050.png)

OTP是没有惟密文攻击，但是有其他攻击。

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%2051.png)

## **Pseudorandom Generators**

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%2052.png)

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%2053.png)

用伪随机key代替随机key。

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%2054.png)

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%2055.png)

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%2056.png)

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%2057.png)

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%2058.png)

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%2059.png)

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%2060.png)

永远不要在加密中使用glibc中的`random()`函数。

## **Negligible vs. non-negligible**

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%2061.png)

可忽略的：意思是比多项式的逆下降地还要快。

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%2062.png)

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%2063.png)

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%2064.png)

## **Attacks on OTP and stream ciphers**

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%2065.png)

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%2066.png)

不能两次使用相同的PRG(k)。

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%2067.png)

客户端到服务端与服务端到客户端应该使用不同的key。

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%2068.png)

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%2069.png)

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%2070.png)

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%2071.png)

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%2072.png)

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%2073.png)

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%2074.png)

中间可以加入p，然后进行篡改。

## **Real-world Stream Ciphers**

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%2075.png)

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%2076.png)

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%2077.png)

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%2078.png)

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%2079.png)

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%2080.png)

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%2081.png)

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%2082.png)

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%2083.png)

## **PRG Security Defs**

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%2084.png)

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%2085.png)

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%2086.png)

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%2087.png)

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%2088.png)

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%2089.png)

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%2090.png)

一个安全的PRG是不可预测的。

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%2091.png)

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%2092.png)

反过来也成立，一个不可预测的PRG是安全的。

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%2093.png)

用到了前面定理(Yao’82)的逆否命题。不安全的PRG是可预测的。

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%2094.png)

更一般的定义，computationally indistinguishable。

## Semantic security

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%2095.png)

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%2096.png)

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%2097.png)

前两个定义都太强了，需要一个弱一些的定义，找到存在的 $m_0$ 与 $m_1$。

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%2098.png)

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%2099.png)

这里给出了semantically secure 的定义。

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%20100.png)

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%20101.png)

OTP是 semantically secure 的。

## **Stream ciphers are semantically secure**

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%20102.png)

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%20103.png)

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%20104.png)

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%20105.png)

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%20106.png)

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%20107.png)

![Untitled](/img/2022-11-21-Dan-Boneh-Cryptography-Week1/Untitled%20108.png)

