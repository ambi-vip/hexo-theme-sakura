---
title: Metronic 学习记录 2
author: Ambi
avatar: 'https://cdn.jsdelivr.net/gh/AmbitionLover/cdn/img/custom/avatar.jpg'
authorLink: 'https://sakura.ambitlu.work/'
authorAbout: 一个好奇的人
authorDesc: 一个好奇的人
comments: true
photos: 'https://keenthemes.com/metronic/themes/metronic/doc/assets/img/demos/demo8.png'
categories: 资源
tags: metronic
description: 这篇文章介绍Metronic KTDatatable如何使用。
abbrlink: 3bef95b3
date: 2020-05-22 10:27:23
keywords:
---

### KTDatatable自定义行内按钮

![](20200522103229.png)
默认下载的html模板中KTDatatable的行内按钮是没有点击事件实例的。官网也没有介绍。
当然我研究出来了。

首先在你需要的点击事件上加上**data-record-id="' + row.OrderID + '" data-ambi-event="delete"**具体如下
```js
<a href="javascript:;" data-record-id="' + row.OrderID + '" data-ambi-event="delete" class="btn btn-sm btn-clean btn-icon" title="删除"></a>
```

```js
datatable.on('click', '[data-record-id]', function() {
    //获取当前对象的record-id。获取什么内容。就写data-xxx。取值就是xxx。
    //Swal.fire($(this).data('record-id'));

    //选择当前是哪个按钮，通过ambi-event判断。
    switch($(this).data('ambi-event')) {
        case 'edit':
            Swal.fire('edit');
            break;
        case 'delete':
            Swal.fire('delete');
            break;
        default:
            Swal.fire('default');
    } 

});
```

### 通过监听hash对页面内容进行加载

```js

"use strict";

// 页面主题
var Ambijs = function() {

	
	// basic demo
	var init = function() {
        //加载页面
		entryPage();
	};

	var hashchange = function() {
        //监听hash
		$(window).on("hashchange", function() {//兼容ie8+和手机端
			entryPage();    //重新加载页面
		});
		
	};

	var entryPage = function() {
		var hash = window.location.hash.split('#')[1];
		var s = KTLayoutHeaderMenu.getMenu();
		if(hash == null){
			hash = 'main.html'
		}
		// 设置当前页位活动页
		s.setActiveItem($('a[href="#'+hash+'"]').closest('.menu-item')[0]);
		// Swal.fire(hash);
		//异步请求加载页面
        $.ajax({
            type: "GET",
            cache: false,
            async:false,    //异步
            url: hash,
            dataType: "html",
            success: function (res) {
				$('#kt_content').html($(res));
            }
        });
	}

	return {
		// public functions
		init: function() {
			init();
			hashchange();
		}
	};
}();

jQuery(document).ready(function() {
	Ambijs.init();
});

```

### KTDatatable的重新加载
有时我们需要根据某些条件更改table的加载。通过查看文档。可以使用[setDataSourceParam](https://keenthemes.com/metronic/?page=docs&section=html-components-datatable)
```js
datatable.setDataSourceParam("deptId",ori.id);
datatable.load();
```


### KTDatatable分页的使用。
有两种方式。
##### 一、是更改KTDatatable的请求参数。
源码在：**src\js\components\datatable\core.datatable.js**

```js
$(datatable.table).siblings('.' + pfx + 'datatable-pager').removeClass(pfx + 'datatable-paging-loaded');

var buildMeta = function() {
	datatable.dataSet = datatable.dataSet || [];
	Plugin.localDataUpdate();
	// local pagination meta
	var meta = Plugin.getDataSourceParam('pagination');
	if (meta.perpage === 0) {
		meta.perpage = options.data.pageSize || 10;
	}
	meta.total = datatable.dataSet.length;
	var start = Math.max(meta.perpage * (meta.page - 1), 0);
	var end = Math.min(start + meta.perpage, meta.total);
	datatable.dataSet = $(datatable.dataSet).slice(start, end);
	return meta;
};
```
通过源码可以看到使用getDataSourceParam来更新查询参数。**我不建议这样的方法，自己多深的水还是懂一些的。**

#### 二、服务器端响应数据更改
参照官网的格式返回，一切ok。表格的分页。排序就全部搞定。
```json
{
    "meta": {
        "page": 1,
        "pages": 35,
        "perpage": 10,
        "total": 350,
        "sort": "asc",
        "field": "ShipDate"
    },
    "data": [***]
}
```
