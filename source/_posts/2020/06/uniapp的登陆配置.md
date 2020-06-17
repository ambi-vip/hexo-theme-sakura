---
title: uniapp的登陆配置
author: Ambi
avatar: 'https://cdn.jsdelivr.net/gh/AmbitionLover/cdn/img/custom/avatar.jpg'
authorLink: 'https://sakura.ambitlu.work/'
authorAbout: 小程序呀
authorDesc: 一个好奇的人
comments: true
photos: 'https://cdn.jsdelivr.net/gh/wang1375830242/PicGo//img/20200604170024.webp'
categories: 技术
description: uniapp可以一到多端开发
abbrlink: 2a04ea56
date: 2020-06-11 22:57:11
tags:
keywords:
---

## 开始

在hibuild新建一个uniapp模板项目。

## 全局工具类

在根目录下新建stone/index.js。插入以下代码。

```js
import Vue from 'vue'
import Vuex from 'vuex'

Vue.use(Vuex)

const store = new Vuex.Store({
	state: {
		hasLogin: false,
		userInfo: {},
    },
    //mutations插件
	mutations: {
		login(state, provider) {

			state.hasLogin = true;
			state.userInfo = provider;
			uni.setStorage({//缓存用户登陆状态
			    key: 'userInfo',  
			    data: provider  
			}) 
			console.log(state.userInfo);
		},
		logout(state) {
			state.hasLogin = false;
			state.userInfo = {};
			uni.removeStorage({  
                key: 'userInfo'  
            })
		}
	},
	actions: {
	
	}
})

export default store

```

### App.vue操作
在App.vue中头部
导入
```vue
import {
	mapMutations
} from 'vuex';
```

在App.vue的onLaun插入

```vue
let userInfo = uni.getStorageSync('userInfo') || '';
if(userInfo.id){
    //更新登陆状态
    uni.getStorage({
        key: 'userInfo',
        success: (res) => {
            console.log(res.data);
            this.login(res.data);
        }
    });
}
```

### 插件注册

在main.js中导入.让其在项目中生效。

```js
import store from './store'
Vue.prototype.$store = store;

const app = new Vue({
    ...App,
	store
})

```

### 登陆文件login.vue

这个``login.vue``文件自己新建，找个自己的喜欢的模板

在
```vue
<script>
	var _this;
	
	import {
	    mapMutations  //引入。下一篇文章会介绍
	} from 'vuex';
	
	export default {
		data() {
	
		},
		mounted() {
			_this= this;
		},
		methods: {
			...mapMutations(['login']), //映射stone的login函数。使其可以通过this.login()使用。
		    async startLogin(){
				
				let userdata={
					"username":this.phoneData,
					"nickname":this.phoneData,
					"id":"123",
				} //保存用户信息
				
				_this.login(userdata);//执行stone的登陆
				
		    }
		}
	}
</script>

```


### 使用登陆者信息

```vue
<script>
	import {
	    mapState,  
	    mapMutations  
	} from 'vuex';  
	export default {
		computed:{
			...mapState(['userInfo']),  //映射使用
		}
	}
</script>


在视图中可直接使用。
<template>
    <text>{{userInfo.username}}</text>
</template>
```