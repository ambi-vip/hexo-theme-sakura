---
title: git 的那些事
author: Ambi
avatar: 'https://cdn.jsdelivr.net/gh/AmbitionLover/cdn/img/custom/avatar.jpg'
authorLink: 'https://sakura.ambitlu.work/'
authorAbout: 一个好奇的人
authorDesc: 一个好奇的人
comments: true
photos: >-
  https://ambi-blog.oss-cn-hangzhou.aliyuncs.com/sakura/img/shallow-focus-photo-of-pink-ceramic-roses-1028707.jpg?x-oss-process=sakura
abbrlink: 3229c6ca
date: 2020-06-07 15:43:14
categories:
tags:
keywords:
description:
---


## 移除Git仓库的node_modules
一般情况下我们是不需要把node_modules提交到Git仓库。如果不小心提交node_modules到git仓库，可以按一下步骤删除仓库的node_modules：

1、在.gitignore文件添加node_modules。避免后续误把node_modules提交到git仓库。

2、按顺序执行以下命令：

```bash
git rm -r --cached node_modules
git commit -m '移除node_modules文件夹'
git push origin master
```

