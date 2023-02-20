---
layout: post
title: 图灵机
categories: CS
description: 
keywords: 形式语言与自动机、图灵机
---

### 图灵机和枚举机的等价性

https://en.m.wikipedia.org/wiki/Enumerator_(computer_science)

![image-20230109114159410](http://pic.inoodles.online/imgimage-20230109114159410.png)

备注：这个证明的第二部分，枚举机E=>图灵机M：对一个枚举机E，若L(E) = L，构造一个图灵机也对应语言L，是很容易理解的；

难点在第一部分：图灵机=>枚举机。之前主要困扰是既然枚举机input为空，如何给其输入s1、s2、s3等等？后面理解到这正是**枚举**一词的精髓，即将所有可能性都列举出来（注意不是每个都属于L），**而这一步其实很容易通过一个图灵机实现（因为列举有明确的规则，与输入无关）**，然后在列举的可能集合里使用图灵机M进行有限步试探（上图中的i），通过这种方式，所有属于L(M)的string最终都会被print，即构造出了我们想要的枚举机