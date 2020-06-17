---
title: jstree 使用记录
author: Ambi
avatar: 'https://cdn.jsdelivr.net/gh/AmbitionLover/cdn/img/custom/avatar.jpg'
authorLink: 'https://sakura.ambitlu.work/'
authorAbout: 一个好奇的人
authorDesc: 一个好奇的人
comments: true
photos: 'https://cdn.jsdelivr.net/gh/AmbitionLover/cdn@v1.1.3/list_8.png'
tags: js
abbrlink: 1d416c62
date: 2020-05-15 14:17:52
categories:
keywords:
description:
---
## jstree默认展开所有子节点
```js
$(‘#div-tree’).jstree({
	 'core': {
       'data': function (node, callback) {
            callback.call(this, data);
        }
     },
})
 .on('ready.jstree',function(){
     $('#div-tree').jstree('open_all')
  });
```

## 获取点击节点的值
```js
$('#kt_tree_4').on("activate_node.jstree", function(e, data) { //#categoryTree是树挂载的元素
    var ori = data.node.original; //original下面有点击节点的数据。
    Swal.fire( ori.id); //获取id
})
```