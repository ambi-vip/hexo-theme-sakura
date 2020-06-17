---
title: Js开发的一些小技巧
author: Ambi
avatar: 'https://cdn.jsdelivr.net/gh/AmbitionLover/cdn/img/custom/avatar.jpg'
authorLink: 'https://sakura.ambitlu.work/'
authorAbout: 一个好奇的人
authorDesc: 一个好奇的人
comments: true
photos: 'https://cdn.jsdelivr.net/gh/AmbitionLover/cdn@v1.1.3/list_7.jpg'
categories: 技术
abbrlink: ed499c51
date: 2020-04-12 19:20:47
tags: js
keywords: js,工具类
description: 开发js真的不在行。这些小技巧，小方法，我觉得挺不错的。整理以下，希望有用
---
### cookie
**JS设置cookie:**

简单方式：document.cookie="name="+username;

```js
//写cookies，一个小时过期 
    function setCookie(name, value) { 
        var exp = new Date(); 
        exp.setTime(exp.getTime() + 60 * 60 * 1000); 
        document.cookie = name + "=" + escape(value) + ";expires=" + exp.toGMTString() + ";path=/"; 
    }
```

注意：
1. path=/，path参数用来设置cookie路径，同一路径下不能存储相同名字的两个cookie，当存第二个的时候会把第一个覆盖其实相当于对第一个进行了赋值操作；  
2. 不同路径下可以存储相同名字的cookie。读取时如果在多个路径下存在多个cookie，则会读取页面所对应的路径（不是物理路径，是cookie的路径）下的cookie，不注意这点  
3. 可能会造成读取的cookie值不正确。删除时只能删除对应路径下的cookie，不指定路径，默认删除的是页面所对应的路径下的cookie。

**读取cookies：**
```js
//读取cookies 
    function getCookie(name) { 
        var arr, reg = new RegExp("(^| )" + name + "=([^;]*)(;|$)"); 
   
        if (arr = document.cookie.match(reg)) 
   
            return unescape(arr[2]); 
        else 
            return null; 
    }
```

**删除cookies：**

```js
//删除cookies 
    function delCookie(name) { 
        var exp = new Date(); 
        exp.setTime(exp.getTime() - 60 * 60 * 1000); 
        var cval = getCookie(name); 
        if (cval != null) 
            document.cookie = name + "=" + cval + ";expires=" + exp.toGMTString() + ";path=/"; 
    }
//删除时只能删除对应路径下的cookie，不指定路径，默认删除的是页面所对应的路径下的cookie。 
```
如果需要设定自定义过期时间、那么把上面的setCookie　函数换成下面两个函数就ok;  

```js
function setCookie(name,value,time) 
{ 
    var strsec = getsec(time); 
    var exp = new Date(); 
    exp.setTime(exp.getTime() + strsec*1); 
    document.cookie = name + "="+ escape (value) + ";expires=" + exp.toGMTString(); 
} 
function getsec(str) 
{ 
    alert(str); 
    var str1=str.substring(1,str.length)*1; 
    var str2=str.substring(0,1); 
    if (str2=="s") 
    { 
        return str1*1000; 
    } 
    else if (str2=="h") 
    { 
        return str1*60*60*1000; 
    } 
    else if (str2=="d") 
    { 
        return str1*24*60*60*1000; 
    } 
}
```


#### 参数求取两位小数。不够的补0

```js
function KeepTwoDecimalFull(num){
	var result = parseFloat(num);
	if(isNaN(result)){
		console.log("参数有误");
		return false;
	}
	result = Math.round(num * 100 ) / 100;
	var s_x = result.toString();
	var pos_decimal = s_x.indexOf('.');

	if(pos_decimal < 0){
		pos_decimal = s_x.length;
		s_x += '.';
	}

	while(pos_decimal < 0){
		pos_decimal = s_x.length;
		s_x += '.';
	}

	while(s_x.length <= pos_decimal + 2){
		s_x += '0';
	}
	return s_x ;
}
```

