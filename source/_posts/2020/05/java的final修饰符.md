---
title: java的final修饰符
author: Ambi
avatar: 'https://cdn.jsdelivr.net/gh/AmbitionLover/cdn/img/custom/avatar.jpg'
authorLink: 'https://sakura.ambitlu.work/'
authorAbout: 博主在默默的学习java
authorDesc: 一个好奇的人
comments: true
photos: 'https://cdn.jsdelivr.net/gh/AmbitionLover/cdn/imges/close-up-of-leaf-326055.jpg'
categories: 技术
tags: java
keywords: java的final修饰符
description: final修饰符是java语言中比较简单的一个修饰符，但也是一个被“误解”较多的修饰符。如何使用final修饰符呢？
abbrlink: 3bcd3b98
date: 2020-05-27 17:36:39
---
### 1. final修饰的变量

#### 1.1 final实例变量
被final修饰的**实例变量**必须显示指定初始值、而且只能在以下的位置指定初始值：
1. 定义final实例变量时指定初始值
2. 在非静态初始化块为final实例变量指定初始值
3. 在构造器中为final指定初始值

本质上final实例变量只能在构造器中被初始化赋值。

#### 1.2 final类变量
对于final类变量而言，同样必须显示指定初始值，而且final类变量只能在**2个**地方制定初始值。
1. 定义final类变量时指定初始值
2. 在静态初始化中为final类变量指定初始值

本质上final类变量只能在静态初始化快中被初始化赋值。

### 2. final 有何用？
```java

class Price{
    final static Price INSTANCE = new Price(2.8);

    //这里加上final。
    static double initPrice = 20;
    double currentPrice ;
    public Price(double discount){
        //根据静态变量计算实例变量
        currentPrice = initPrice - discount;
    }
}
public class PriceTest{
    public static void main(String[] args) {
        //通过Price的INSTANCE访问currentPrice实例
        System.out.print(Price.INSTANCE.currentPrice);
    }
}
//不用final修饰initPrice的运行结果
-2.8
//initPrice前面用final修饰的运行结果
17.2
```
产生这样的结果时在使用final变量修饰类变量时，如果定义该final变量指定了初始化，而且该初始值在编译时就被确定了。系统将不会在静态初始化块中对该类变量赋初始值，而将是在类定义中直接使用该初始化值替代该final变量。类似于**宏变量。**
### 2.1 final 的“宏变量”
```java
public class PriceTest{
    public static void main(final String[] args) {
        //下面定义4个final“宏变量”
        final int a = 5 + 2;
        final double b = 3 / 2;
        final String str = "Ambi" + "java";
        final String book = "java" + "讲义";

        //下面的book2变量的值因为调用了方法，所以无法在编译的时候被确定下来。
        final String book2 = "java" + String.valueOf("讲义");

        System.out.println(book == "java讲义");
        System.out.println(book2 == "java讲义");
    }
}
//运行结果
true
false
```
对于final实例变量来说，只有在**定义该变量时指定初始化值**才有“宏变量”效果。示例如下：
```java


public class PriceTest{
    //定义3个final变量
    final String str1;
    final String str2;
    final String str3 = "java";
    //str1,str2分别在非静态初始化块、构造器中初始化
    {
        str1 = "java";
    }
    public PriceTest(){
        str2 = "java";
    }
    //p判断str1,str2,str3是否宏替换
    public void display(){
        System.out.println(str1 + str1 == "javajava");
        System.out.println(str2 + str2 == "javajava");
        System.out.println(str3 + str3 == "javajava");
    }
    public static void main(String[] args) {
        PriceTest p = new PriceTest();
        p.display();
    }
}
//运行结果
false
false
true
```