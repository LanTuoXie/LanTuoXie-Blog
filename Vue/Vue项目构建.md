# 构建Vue项目

## 按照官网教程安装

```bash
//先安装脚手架
cnpm i -g vue-cli

//查看项目目标列表: webpack browserify pwa 等项目模板
vue list

//使用webpack模板创建项目
vue init webpack my-project

//进去新创建的文件夹
cd my-project

//打开package.json 看scripts字段
//dev 可以使用npm run dev 或者 npm start 开启开发模式
//build 可以使用npm run build 打包项目，生成最终
//还有其他的命令根据创建项目的时候的选择决定

```

webpack模板默认安装了`vue`、`vue-router`、`postcss`、`babel`，如果需要其它的可以自己再安装。

## 安装sass

```bash
cnpm i --save-dev node-sass sass-loader
```

安装好后就可以在`.vue`文件style便签设置 `<style scoped lang="scss">`就可以开启scss功能

## 安装axios

```bash
cnpm i --save axios
```

在`main.js`文件中引入`axios`并将其指定为`Vue.prototype.$http = axios`

```js
//main.js
//全局的Vue，所有组件都是这个Vue的实例
import Vue form 'vue';
import axios from 'axios';

//由于所有Vue的组件都继承于这个全局的Vue，所有的组件都能够通过$http拿到axios
//自定义全局属性要带'$'
Vue.prototype.$http = axios;

```

## 安装vuex

```bash
cnpm i --save vuex
```

在`main.js`文件中引入`Vuex`，使用`Vue.use(Vuex)`且创建`store`，然后将`store添加到全局 new Vue({store})`

```js
import Vue from 'vue'
import Vuex from 'vuex'
import App from './App'

Vue.use(Vuex)

const store = new Vuex.Store({
  state: {
    count: 0
  }  
})

new Vue({
  el: '#app',
  component: App,
  store,
  template: "</App>"
})
```

## 项目结构

- `/src`
  - `/assets`
  - `/components`
  - `/containers`
  - `/modules`
  - `/router`
  - `/scss`
  - `App.vue`
  - `main.js`

**/assets:**

主要放`图片`、`字体`等静态文件

**/components:**

`公用组件`，所有页面都可以公用的组件，或者单例组件，像`弹出窗`、`按钮`、等可以多次复用的组件

**/containers:**

`页面组件`，`router`中的路由组件，`相对路由`来划分这个文件夹的结构<br>
比如路由`/user/change_psw` 和`/login`

- `/containers`
  - `User.vue`
  - `/user`
    - `ChangePsw.vue`
  - `Login.vue`

```js
import User from '@/containers/User';
import ChangePsw from '@/containers/user/ChangePsw';
import Login from '@/containers/Login';

const router = [
  {
    // 匹配 /user
    path: '/user',
    component: User,
    children: [
      {
        // 匹配 /user/change_psw
        path: 'change_psw',
        component: ChangePsw
      }
    ]
  }, {
    // 匹配 /login
    path: '/login',
    component: Login
  }
]
```

**/modules:**

`Vuex`的状态管理，这里按每个页面划分，当然还可以别的方法，这里只供参考<br>

- `/modules`
  - `modules.js`
  - `index.js`
  - `user.js`

`modules.js`负责将所有模块收集起来(类似reduce的合成)，`index.js`和`user.js`分别管理`页面index和页面user的全局状态`

```js
//modules.js
import index from './index'
import user from './user'

export default {
  index,
  user
}
```

```js
//index.js
export default {
  namespaced: true,
  state: {
    a: 1
  }
}
```

```js
//user.js
export default {
  namespaced: true,
  state: {
    a: 2
  }
}
```

除了这些模块的状态，还有一个全局的状态，如果全局的状态有点大，可以将其独立成一个文件。<br>
若超大，再开文件夹，再细化。同理，页面状态也可以。<br>
最后在`mian.js`装载这些模块

```js
import modules from '@/modules/modules';
import Vuex from 'vuex';
import Vue from 'vue';

const store = new Vuex.Store({
  modules,
  // ...定义全局的状态
})

new Vue({
  store,
  // ... 其他的配置
})
```

**/router:**

如果项目比较大，可以按模块来划分这个文件夹的结构。小项目，一个文件即可。

- `/router`
  - `buy-ticket.js`
  - `index.js`
  
和别的文件一样，`index.js`是所有模块路由的总和，`buy-ticket.js`是买票模块的路由 <br>
由于vue的路由是可以嵌套的，我们完全可以拆分路由，然后再合并。

```js
//buy-ticket.js

export default [
  {
    //路由配置
  },
  {
    //路由配置
  }
]
```

```js
//index.js 借助数组扩展符 或者 成为嵌套路由children的值

import Vue from 'vue';
import Router from 'vue-router';
import buyTicketRouters from '@/router/buy-ticket';
import Index from '@/containers/Index';
import Ticket from '@/containers/Ticket';

Vue.use(Router);

export default new Router({
  routes: [
    {
      path: '/',
      components: Index
    },
    ...buyTicketRouters,
    //采用children
    {
      path: '/ticket'
      components: Ticket,
      children: buyTicketRouters
    }
  ]
})
```

具体项目，要按项目的功能来拆分，以上只供参考。

**/scss:**

这个样式项目结构，一言难尽，可以了解我的scss相关的文章

[Sass项目结构之 7-1模式](http://www.cnblogs.com/lantuoxie/p/8682546.html)
