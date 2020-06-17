---
title: Mysql知识点整理
author: Ambi
avatar: 'https://cdn.jsdelivr.net/gh/AmbitionLover/cdn/img/custom/avatar.jpg'
authorLink: 'https://sakura.ambitlu.work/'
authorAbout: 一个好奇的人
authorDesc: 一个好奇的人
comments: true
photos: 'https://cdn.jsdelivr.net/gh/AmbitionLover/cdn@v1.1.3/list_3.jpg'
categories: 技术
abbrlink: f1c7d673
date: 2020-04-09 13:10:20
tags:
keywords:
description:
---
### 常用函数
#### 字符函数
    1.length 获取参数值的直接个数
    SELECT	LENGTH('Ambi')
    SELECT LENGTH('樱花庄')

    SHOW VARIABLES LIKE '%char%'

    2.concat 拼接字符串
    SELECT CONCAT ('a','_','b')

    3.upper、LOWER
    SELECT UPPER('zs')
    SELECT LOWER('AMBI')

    4.substr、SUBSTRING 
    索引从1开始
    SELECT SUBSTR('我爱上了你',2) AS out_put
    截取制定索引处制定长度字符
    SELECT SUBSTR('我爱上了你',2,3) AS out_put

    5.instr
    返回字串在主第一次出现的索引
    SELECT INSTR('今天天气正好','天气') AS out_put

    6.trim
    SELECT LENGTH(TRIM('   Ambi   ')) AS out_put
    SELECT TRIM('a' FROM 'aaaaaaaAmbiaaaaaa') AS out_put

    7.lpad 用指定字符实现左填充指定长度。RPAD 右填充
    SELECT LPAD('Ambi',10,'*') AS out_put

    8.replace 替换
    SELECT REPLACE("AAAABAA","A",'B')
#### 数学函数
    1.round 四舍五入
    SELECT ROUND(-1.66)

    2. CEIL 向上取整 FLOOR 向下取整
    SELECT CEIL(-1.02)

    3.truncate 截断
    SELECT TRUNCATE(1.69999,1)

    4. MODE 取余 
    MOD(a,b) : a-a/b*b
    SELECT MOD(10,-3)
    SELECT 10%3
#### 日期函数
    1.now 返回当前系统日期+时间
    SELECT NOW();

    2.curdata 返回当前系统日期，不包含时间
    SELECT CURDATE();

    3. CURTIME 返回当前时间,不包含日期
    SELECT CURTIME();

    4.可以获取指定部分
    SELECT YEAR(NOW()) 年;
    SELECT MONTH(NOW()) 月;
    SELECT MONTHNAME(NOW()) 英文月;

    5.str_to_date 
    SELECT STR_TO_DATE('2020/4/9','%Y/%c/%d') AS out_put

    6.date_format 将日期转换为字符
    SELECT DATE_FORMAT(NOW(),'%y年%m月%d日') AS out_put

    7.DATEDIFF 两个时间之间的天数
    SELECT DATEDIFF(NOW(),'1998-12-9');
### 分组函数
sum、avg、max、min、count