---
title: 开发springCloud遇到的坑
author: Ambi
avatar: 'https://cdn.jsdelivr.net/gh/AmbitionLover/cdn/img/custom/avatar.jpg'
authorLink: https://sakura.ambitlu.work/
authorAbout: 2020最新的Cloud技术
authorDesc: 2020最新的Cloud技术
comments: true
photos: 'http://image.bygit.cn/list_12.png'
categories: 技术
keywords: SpringCloud
abbrlink: cead5908
date: 2020-03-18 00:26:33
tags:
description: 总结我在开发SpringCloud的经验
---

##### 不配置数据源启动BOot。

```java
Disconnected from the target VM, address: '127.0.0.1:64057', transport: 'socket'
```
原因：我的项目中使用了Alibaba的**druid-spring-boot-starter**。

解决方案:去掉这个依赖。

##### 不配置数据源启动BOot。