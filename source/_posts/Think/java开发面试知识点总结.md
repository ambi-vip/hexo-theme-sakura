---
title: java开发面试知识点总结
author: Ambi
avatar: 'https://cdn.jsdelivr.net/gh/AmbitionLover/cdn/img/custom/avatar.jpg'
authorLink: 'https://sakura.ambitlu.work/'
authorAbout: 一个好奇的人
authorDesc: 一个好奇的人
comments: false
photos: 'http://image.bygit.cn/list_11.png'
categories: 随想
description: 看完大量得资料后，总结自己得知识点体系
abbrlink: 448646fd
date: 2020-03-24 15:37:11
tags:
keywords:
---

[常见得知识点](https://juejin.im/post/5e18879e6fb9a02fc63602e2#heading-10)

[理解反射](https://blog.csdn.net/sinat_38259539/article/details/71799078)

#### 元数据和永久代
在 JDK 1.8中移除整个永久代，取而代之的是一个叫元空间（Metaspace）的区域（永久代使用的是JVM的堆内存空间，而元空间使用的是物理内存，直接受到本机的物理内存限制）。
1. 字符串存在永久代中，容易出现性能问题和内存溢出。
2. 类及方法的信息等比较难确定其大小，因此对于永久代的大小指定比较困难，太小容易出现永久代溢出，太大则容易导致老年代溢出。
3. 永久代会为 GC 带来不必要的复杂度，并且回收效率偏低。
4. Oracle 可能会将HotSpot 与 JRockit 合二为一。


[如何正确的将数组转换为ArrayList](https://gitee.com/SnailClimb/JavaGuide/blob/master/docs/java/Java%E7%96%91%E9%9A%BE%E7%82%B9.md#11-%E6%AD%A3%E7%A1%AE%E4%BD%BF%E7%94%A8-equals-%E6%96%B9%E6%B3%95)