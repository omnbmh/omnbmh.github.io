---
layout: post
title: SpringBoot+Vue实战（四）- 使用 Vue 开发登录页面
date: 2019-08-04 20:00:00
tags: [Spring Boot, Vue]
categories: [SpringBoot+Vue实战]
baseline:
artileno: 20190804_01
---

回顾一下[第一篇](https://omnbmh.github.io/2019-08-01/springboot-vue-start.html)中我们已经生成一个默认的Vue项目,在本篇基于生成的项目，来开发一个登录页面。

* 0x01 安装依赖

```
cd cobra-vue
npm install element-ui
npm install axios
npm install vuex
```

* 0x02 打开 `main.js` 引入 ElementUI

```
import ElementUI from 'element-ui'
Vue.use(ElementUI)
```
> [查看源码](https://github.com/omnbmh/cobra/blob/master/cobra-vue/src/main.js)

* 0x03 在 `src/components` 目录新增 `Login.vue`
> [查看源码](https://github.com/omnbmh/cobra/blob/master/cobra-vue/src/components/Login.vue)

* 0x04 编写 api.js 工具类

项目中会用到大量的ajax请求，为了便于处理状态、异常等，我们将将封装一个js `src/utils/api.js` 来统一处理，主要实现了 get post 请求

> [查看源码](https://github.com/omnbmh/cobra/blob/master/cobra-vue/src/utils/api.js)


* 0x05 在 `main.js` 中 导入 api 将 get post 方法,添加到 prototype中
```
import {getRequest} from './utils/api'
import {postRequest} from './utils/api'

Vue.prototype.getRequest = getRequest
Vue.prototype.postRequest = postRequest
```

* 0x06 在登录功能中，使用了Vuex的store模块，编写 store 中变量和方法 新建 `src/store/index.js`

```
import Vue from 'vue'
import Vuex from 'vuex'

Vue.use(Vuex)


export default new Vuex.Store({
    state:{
        user:{}
    },
    mutations:{
        login(state,user){
            state.user = user;
            window.localStorage.setItem('user',JSON.stringify(user))
        }
    },
    actions:{}
});
```
> [查看源码](https://github.com/omnbmh/cobra/blob/master/cobra-vue/src/store/index.js)

代码中 login 方法是成功登录后 将当前用户信息写入 storage

* 0x07 配置路由 `router/index.js` 访问根目录时展示登录页面

```
import Vue from 'vue'
import Router from 'vue-router'
import HelloWorld from '@/components/HelloWorld'
import Login from '@/components/Login'

Vue.use(Router)

export default new Router({
  routes: [
    {
      path: '/',
      name: 'Login',
      component: Login,
      hidden: true
    },{
      path: '/home',
      name: 'Home',
      component: HelloWorld,
      hidden: true,
      meta:{
        requireAuth: true
      }
    }
  ]
})

```

代码中配置了两个路由 访问 `/` 展示登录页面 成功登录后跳转到 `/home`

* 0x08 开发环境配置

因为前端页面和后端端口不同 为方便开发 需要端口转发 修改 `config/index.js`，找到 proxyTable 添加下面的内容

```
proxyTable: {
  '/':{
    target:'http://127.0.0.1:9999',
    changeOrigin: true,
    pathRewrite:{'^/':''}
  }
},
```

* 0x09 验证一下，启动后台服务 启动前端页面 `npm run dev`

```
ONE  Compiled successfully in 2328ms                                                                                                                                    下午4:26:03

 I  Your application is running here: http://localhost:8080
```

访问一下 http://localhost:8080

![登录页面]({{site.url}}/assets/20190804_01_01.png)

登录成功后，跳转

![首页]({{site.url}}/assets/20190804_01_01.png)

#### 问题
* 1. 登录老是 cors 跨域问题

```
修改后台代码 WebSecurity formLogin
```

* 2. 老是报错 `_this.$store.commit` 没有 commit 属性

```
在 main.js 中 new Vue 的时候，加上  store
```

#### 参考资料:
> Vue.js 教程 https://www.runoob.com/vue2/vue-tutorial.html
