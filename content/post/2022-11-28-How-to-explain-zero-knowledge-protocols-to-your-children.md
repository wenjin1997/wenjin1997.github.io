---
layout:     post
title:      "How to explain zero-knowledge protocols to your children"
# subtitle:   ""
# excerpt: "Zero Knowledge Proofs:An illustrated primer"
description: "文章《How to explain zero-knowledge protocols to your children》的笔记，通过阿里巴巴与大盗的故事解释什么是零知识证明。"
date:       2022-11-28
author:     "谢文进"
image: "/img/2022-11-28-How-to-explain-zero-knowledge-protocols-to-your-children/background.jpg"
published: 2022-11-28
tags:
    - ZKP
    - 笔记 

categories: [ ZKP ]
URL: "/2022/11/28/How-to-explain-zero-knowledge-protocols-to-your-children/"
---

在文章 **[How to explain zero-knowledge protocols to your children](http://pages.cs.wisc.edu/~mkowalcz/628.pdf)** 中通过阿里巴巴与神秘的洞穴的故事介绍了什么是零知识证明。中文翻译文章见[【译】How to Explain Zero-Knowledge Protocols to Your Children](https://blog.dreamerryao.wiki/archives/%E8%AF%91howtoexplainzero-knowledgeprotocolstoyourchildren)。下面简要记录文章的笔记：

## The Strange Cave of Ali Baba

阿里巴巴每天都去集市上，总被小偷偷东西，每次小偷都跑到一个洞穴，里面有一个岔路口，有两条道路，阿里巴巴追着小偷来到洞穴，阿里巴巴每次只能选择一个口进入，每次小偷都能逃走。在经历悲催的40次被偷后，不科学呀，每次小偷都能选择阿里巴巴不走的那条路，那小偷得多幸运，$\frac{1}{2^{40}}$的概率，不太可能！！！洞穴一定有什么不可告人的秘密！

![Untitled](/img/2022-11-28-How-to-explain-zero-knowledge-protocols-to-your-children/Untitled.png)

一天阿里巴巴提前藏在洞穴里，发现小偷说出了“芝麻开门”咒语，洞穴连接了起来。

![Untitled](/img/2022-11-28-How-to-explain-zero-knowledge-protocols-to-your-children/Untitled%201.png)

难怪这些小偷每次能顺利逃脱。后来阿里巴巴不断实验咒语，发现可以修改咒语，现在只有他知道新的咒语了。阿里巴巴把这个神奇的经历写在手稿上流传了下来。

## The Fate of the Manuscript

后来，阿里巴巴的后代 Mick Ali 知道咒语的秘密，但是他不想告诉世人这个秘密，只想说服别人相信他知道这个秘密。一家电视台独家报道这个过程，实验是这样进行的：

1. 摄影团队先拍摄两个死胡同的细节
2. 每个人都离开洞穴，Mick Ali 单独一人进入洞穴
3. 记者进入到洞穴的岔路口，抛一枚银币，选择左边还是右边，接着大喊让Mike从选择的那一边出来
4. 重复上述过程40次

Mick连续成功40次，已经能足够说服我们他知道咒语。

## The Jealous Reporter

嫉妒的记者找来一个演员装扮成Mick的样子，但是他不知道咒语，记者也模拟上述过程，不过最后剪掉失败的片段，只展示连续成功40次的结果。

模拟的情况和真实的情况一同向世人进行展示，人们也无法区别孰真孰假，这不恰好说明咒语的秘密没有泄露嘛！但是人们都相信Mick知道咒语的秘密。这一过程就在证明**零知识**！！！

## The Tests in Parallel

上面的证明过程要连续进行40次，有没有方法并行呢？可以建造这样一座大厦，每一层有一个洞穴，每一个洞穴都有自己的咒语，每个洞穴有一个演员，在同一时刻他们进入洞穴然后随机选择进入哪一边，最后出来。但是这样的话，需要在秘密的数量和拍摄场景数量上做一个平衡。

## The Prior Agreement

之前的模拟场景需要后期剪掉那些不成功的场景，有没有连续进行挑战成功的办法呢？那就是验证者和证明者之间事先商定随机选择走哪边，连续进行40次也能挑战成功。

## A Single Test, A Single Secret

如何只进行一次测试，就让人们足够相信拥有秘密呢？建造下面这样的洞穴：

![Untitled](/img/2022-11-28-How-to-explain-zero-knowledge-protocols-to-your-children/Untitled%202.png)

只需要单独一次测试就能够让人达到连续40次测试相同水平的信服度，来相信Mick是知道这个咒语的秘密的。