---
title: 'Metronic 学习记录(1) '
author: Ambi
avatar: 'https://cdn.jsdelivr.net/gh/AmbitionLover/cdn/img/custom/avatar.jpg'
authorLink: 'https://sakura.ambitlu.work/'
authorAbout: 一个好奇的人
authorDesc: 一个好奇的人
comments: true
photos: 'https://keenthemes.com/metronic/themes/metronic/doc/assets/img/demos/demo1.png'
categories: 资源
description: Metronic是一个非常优秀的前端框架。市面上没有教程，于是自己研究了~~~~
abbrlink: b68e9a35
date: 2020-05-10 11:36:14
tags: metronic
keywords:
---
## 序言

下载：https://share.weiyun.com/n5eIQE0p

我百度了几天，都是没有教程的。于是自己研究了~~~
[中文版网站](http://metronic.net.cn/)
[官网](https://keenthemes.com/metronic/)

Metronic是一个非常优秀的前端框架。所以设计思路很优秀、集成非常多的插件、界面美丽。类似于程序设计一般的思路。有html、vue、react、angular、laravel版本。可以帮你“快速制作成想要的系统？”

**需要的小伙伴联系我即可。我的是最新的版的(7.0)**

来自官网的介绍:

Metronic built with HTML, SASS, jQuery, Bootstrap 4 and Angular 8 and it can be used to build any web application regardless used server side languages and technologies. To run the Metronic build tasks you will need to install nodejs, npm, bower, angular-cli and others.

首先了解下文件的结构。这里放出官网的。
[在这里查看官网的文件结构](https://keenthemes.com/metronic/?page=docs&section=html/files-structure)

## 接下来以html为例介绍。

打开theme>default>demo1 可以看到有两个文件夹 。src和tools。

src是网站的源码,不可直接运行。tools是对源码进行打包管理文件。

[按照这个方法对源文件进行打包](https://keenthemes.com/metronic/?page=docs&section=html/webpack-quick-start)
打包也就是安装对于依赖，sass编译。js压缩等一系列操作。

打包完成后会出现dist文件。也就是
![20200510115513.png](20200510115513.png)

源文件的每个html页面都是可以独立运行。我想要的要的效果是。主页面加载公共部分，如：菜单、工具、全局设置等。页面主体可根据
菜单的点击进行加载。这样的好处是菜单等公共部分在使用中只加载一次。**于是开始我的改造**

在src->assets->js->global->layout 文件夹下是全局的布局配置。实现的技术叫ajax异步加载。不用frame(我试过了。效果不行，一大堆问题)
无需自己写js文件，在我查看源码之后。其实在layout.js中已经有对菜单做处理的代码。而且还有demo。
![](20200510125054.png)

在这个init函数中加上我们需要的代码
```js
//给所有菜单按钮加上点击事件
$('#kt_aside_menu, #kt_header_menu').on('click', '.kt-menu__link[href!="javascript:;"]', function(e) {
    //获取菜单按钮的url
    var dataUrl = $(this).attr('href');
    var p =  this.parentNode ;
    //模拟演示。
    swal.fire(dataUrl);
    //设置这个按钮为活动
    asideMenu.setActiveItem(p);
    e.preventDefault();
});
````

asideMenu是KTMenu的对象，KTMenu在menu.js（找不到就行全局搜索，这是个js文件）中可以看到。setActiveItem函数会先加载Plugin.resetActiveItem();对菜单进行所有活动的取消。
![](20200510125610.png)

仿照模板js文件的写法。写一个自己的js文件。并加载到index.html中
```js
'use strict';
// Class definition


//主页文件加载出来
var CountAll = function() {
    // demo initializer
    var start = function(){
        console.log("INit ");
        //异步请求加载页面
        $.ajax({
            type: "GET",
            cache: false,
            async:false,    //异步
            url: ctx + "system/main",   //改成可以访问的页面
            dataType: "html",
            success: function (res) {
                $('#kt_content').html($(res));
            }
        });
        return;
    }
    return {
        // Public functions
        init: function() {
            // init dmeo
            start();
        },
    };
}();


jQuery(document).ready(function() {
    CountAll.init();
});

```
这个时候把id=“kt_content”的div的内容转移到main.html中。head和index.html一样。

**注意**   main.html中不能在包含一下俩个文件。这两个是全局加载文件。因为index.html已经加载过。main.html如果有。加载到index就会出现按钮没有反应的现象。导致已经加载选项失效。
    <!--begin::Global Theme Bundle(used by all pages) -->
    <script src="assets/plugins/global/plugins.bundle.js" type="text/javascript"></script>
    <script src="assets/js/scripts.bundle.js" type="text/javascript"></script>
    
### 如何使用正确登陆页面

***
最新版本的（7.0）没有以前的form.ajaxSubmit。因此手动写ajax请求即可。
```js

$('#kt_login_signin_submit').on('click', function (e) {
    e.preventDefault();
    var btn = $(this);
    var form = $(this).closest('form');
    var username = $("input[name='username']").val();
    var password = $("input[name='password']").val();
    var validateCode = $("input[name='validateCode']").val();
    var rememberMe = $("input[name='rememberMe']").is(':checked');
    // var rememberMe = $("#rememberMe").is(':checked');
    validation.validate().then(function(status) {
        if (status == 'Valid') {
            btn.addClass('kt-spinner kt-spinner--right kt-spinner--sm kt-spinner--light').attr('disabled', true);
            $.ajax({
                type: "post",
                cache: false,
                async:false,    //异步
                url: "http://localhost:84/login",
                data: {
                    "username": username,
                    "password": password,
                    "validateCode": validateCode,
                    "rememberMe": rememberMe
                },
                success: function (res) {
                    
                    btn.removeClass('kt-spinner kt-spinner--right kt-spinner--sm kt-spinner--light').attr('disabled', false);
                    
                }
            });
            // swal.fire({
            //     text: "All is cool! Now you submit this form",
            //     icon: "success",
            //     buttonsStyling: false,
            //     confirmButtonText: "Ok, got it!",
            //     confirmButtonClass: "btn font-weight-bold btn-light-primary"
            // }).then(function() {
                
            // });
        } else {
            swal.fire({
                text: "抱歉，似乎检测到一些错误，请重试。",
                icon: "error",
                buttonsStyling: false,
                confirmButtonText: "Ok, got it!",
                confirmButtonClass: "btn font-weight-bold btn-light"
            }).then(function() {
                KTUtil.scrollTop();
            });
        }
    });
    });
```

### 日期转换
```js
//处理日期函数
function setDate(str){
    var dateArr = [
        'Jan',
        'Feb',
        'Mar',
        'Apr',
        'May',
        'Jun',
        'Jul',
        'Aug',
        'Sep',
        'Oct',
        'Nov',
        'Dec'];

    var arr = str.split(' ');
    var tmp = dateArr.indexOf(arr[0]);
    return (tmp+1) + '月' + arr[1] +'日';
}
```