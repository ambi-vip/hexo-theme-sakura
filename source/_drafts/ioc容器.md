---
title: ioc容器
author: Ambi
avatar: 'https://cdn.jsdelivr.net/gh/AmbitionLover/cdn/img/custom/avatar.jpg'
authorLink: 'https://sakura.ambitlu.work/'
authorAbout: 一个好奇的人
authorDesc: 一个好奇的人
comments: true
photos: 'https://cdn.jsdelivr.net/gh/AmbitionLover/cdn@v1.1.3/list_8.png'
abbrlink: 9b1f652a
date: 2020-05-29 18:47:13
categories:
tags:
keywords:
description:
---
## ioc容器bean管理xml注入空值和特殊符号

### set方法注入
需要对应实体类是set方法
```xml
<bean id="User" class="com.company.User">
    <property name="bname" value="值"></property>
</bean>
```

### 空值

```xm
<bean id="User" class="com.company.User">

        <property name="bname" value="值"></property>
        <property name="address"><null/></property>

    </bean>
```