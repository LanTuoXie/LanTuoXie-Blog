#Vuex基本概念
- `State`
- `Getter`
- `Mutation`
- `Action`
- `Module`

简单的Store
```js
import Vue from 'vue';
import Vuex from 'vuex';

Vue.use(vuex);

const store = new Vuex.Store({
  state: {
    count: 0
  },
  mutations: {
    increment (state) {
      state.count++;
    } 
  }
});

store.commit('increment');
console.log(store.state.count);// 1

```

**常见流程**

`Vue Component` --dispatch--> `Action` <br>
`Action`  --commit--> `Mutations` <br>
`Mutations` --mutate-->   `State` <br>
`State` --render--> `Vue Component` <br>

##State
**获取vuex中的状态方法**

在`计算属性:computed`中返回某个状态，要获取Vuex那个状态，要在`computed`中定义<br>
由于在全局使用了`Vue.use(Vuex)`，所有组件可以通过`this.$store`拿到Vuex的`store`<br>
```js
const Counter = {
  template: `<div>{{count}}</div>`,
  computed: {
    count () {
      return this.$store.state.count
    }
  }
}
```
由于`computed的属性是函数`，那么在返回状态之前，还有更多操作的空间。

**mapState辅助函数**
```js
import {mapState} from 'vuex'
```

`mapState`辅助函数接收一个`对象`或者一个`字符串数组`，返回`计算属性:computed`<br>
```js
//这个可以单独放在一个文件中，重复使用
const vuexComputed = mapState({
  //使用函数一定要有返回值

  //箭头函数写法，第一参数是state
  count: state => state.count,
  
  //字符串形式 'count' 等同于 state => state.count
  countAlias: 'count',
  
  //可以使用'this'获取局部状态，使用常规函数，'this'会绑定为 computed的组件实例
  countPlusLocalState (state) {
    return state.count + this.localCount;
  }
});

```

然后将mapState生成的vuexComputed和Vue组件实例的computed合并<br>
使用对象展开运算符`...`
```js
{
  //...某个组件
  
  computed: {
    localComputed () {},
    //使用对象展开运算符将vuexComputed混入到外部对象中
    ...vuexComputed
  }
}
```

如果状态都只是单纯的显示，可以传一个`字符串数组`给`mapState`
```js
const vuexComputed = mapState(['count']);
```

完整的demo:
```js
//main.js
import Vue from 'vue'
import App from './App'
import router from './router'
import Vuex from 'vuex'
import axios from 'axios'

Vue.use(Vuex)

const store = new Vuex.Store({
  state: {
    count: 0
  },
  mutations: {
    increment (state) {
      state.count++
    }
  }
})

store.commit('increment')
console.log(store.state.count)

Vue.config.productionTip = false
Vue.prototype.$http = axios

/* eslint-disable no-new */
new Vue({
  el: '#app',
  store,
  router,
  components: { App },
  template: '<App/>'
})
```

`main.js`要使用`Vue.use(Vuex)`以及将`store`添加到全局`new Vue({store})`<br>
下面的`所有子组件`才可以拿到`store`<br>

```html
<template>
  <div class="hello">
    <div>{{count}}</div>
    <div>{{countAlias}}</div>
    <div>{{countPlusLocalState}}</div>
  </div>
</template>
```

```js
//Component HelloWorld
import {mapState} from 'vuex'

const vuexComputed = mapState({
  count: state => state.count,
  countAlias: 'count',
  countPlusLocalState (state) {
    return state.count + this.localCount
  }
})

export default {
  name: 'HelloWorld',
  data () {
    return {
      localCount: 2
    }
  },
  computed: {
    ...vuexComputed
  }
}
```

##Getter
`Getter`和`计算属性:computed`功能一样，只不过，它是`store`的，也会缓存计算的值。<br>

**组件中拿到getter**

`Getter`会暴露`store.getters`对象。
```js
const store = new Vuex.Store({
  state: {
    count: 0,
    arr: [1, 2, 3, 4, 5, 6]
  },
  mutations: {
    increment (state) {
      state.count++
    }
  },
  getters: {
    get6: state => {
      return state.arr.filter(num => num === 6)
    }
  }
})
```

```js
{
  //...某个组件
  computed: {
    get6 () {
      return this.$store.getters.get6
    }
  }
}
```

在定义`getter`时，有两个参数传入`state`、`getters`
```js
getters: {
  get6: state => state.arr.filter(num => num === 6),
  getNum: (state, getters) => {
    return getters.get6
  }
}
```

**定义getter方法**
```js
//要使用函数要封装2层函数以上，在组件中拿到getTodoById时，默认执行第一层
getters: {
  getTodoById: state => id => state.todos.find(todo => todo.id === id)
}

store.getters.getTodoById(2)
```

**mapGetters**
功能：将`store`中的`getter`映射到局部的计算属性`computed`中<br>
和`mapState`类似，支持传入`对象`或者`字符串数组`，不过只能获取，不能重写，或者定义<br>

```js
//store
{
  state: {
    arr: [1, 2, 3, 4, 5, 6, 7, 8]
  },
  getters: {
    get6: state => {
      return state.arr.find(num => num === 6)
    },
    getNum: state => num => state.arr.find(n => n === num) || '错误num'
  }
}
```

```html
<template>
  <div class='helloworld'>
    <div>{{get6}}</div>
    <div>{{getNum(4)}}</div>
  </div>
</template>
```

```js
const vuexGetter = mapGetters(['get6', 'getNum']);

//如果想定义别名使用对象的方式
//const vuexGetter = mapGetters({getSix: 'get6', getNumber: 'getNum'})

export default {
  name: 'HelloWorld',
  data () {
  },
  computed: {
    ...vuexGetter
  }
}
```

##Mutation
修改`Vuex`的`store`中的状态的唯一方式是`commit mutation`。<br>
每个`mutation`都有一个字符串的**事件类型(type)**和一个**回调函数(handler)**<br>
```js
const store = new Vuex.Store({
  state: {
    count: 1
  },
  mutations: {
    increment (state) {
      // 变更状态
      state.count++
    }
  }
})
```

```js
//type: increment
store.commit('increment')
```

**Payload**
```js
mutations: {
  increment (state, n) {
    state.count += n
  }
}
store.commit('increment', 10)
```

`store.commit`可以支持`多参数模式`，或者`一个对象模式`
```js
store.commit({
  type: 'increment',
  amount: 10
})
```
其中`一个对象模式`和`redux`很像<br>
而且事件类型type也可以使用常量来替代
```js
// mutation-types.js
export const SOME_MUTATION = 'SOME_MUTATION';

// store.js
import Vuex from 'vuex';
import { SOME_MUTATION } from './mutation-types';

const store = new Vuex.Store({
  state: { ... },
  mutations: {
    // 我们可以使用 ES2015 风格的计算属性命名功能来使用一个常量作为函数名
    [SOME_MUTATION] (state) {
      // mutate state
    }
  }
})
```

**组件中提交mutation**

在组件中可以通过`this.$store.commit('xxx')`提交`mutation`<br>
还有就是使用`mapMutations`辅助函数，不过这次不是映射到计算属性，而是`methods`<br>
使用方式和`mapGetters`一模一样
```js
import { mapMutations } from 'vuex';

//对象的方式可以重新命名，字符串数组是直接引用
const vuexMutations = mapMutations(['increment']);

export default {

  methods: {
    ...vuexMutations
  }
}

```

**Mutation 和 Reduce**

`Reduce`的基本形式
```js
(state, Action) => newState || state
```
转化为`Mutation`，那么
```js
const Action = {
  type: 'increment',
  amount: 10
}

const initState = {
  count: 0
}

export default (state = initState, Action) => {
  switch(Action.type){
    'increment':
      return Object.assign({},state,{count: Action.amount + state.count})
    default:
      return state
  }  
}
```

在`redux`中改变状态：<br>
`store.dispatch(Action) -> reduce(state, Action) -> newState -> render view`

在`mutation`中改变状态:<br>
`store.commit('increment', playload) -> mutations['increment'](state, playload) -> newState -> render view`

##Action
一般`Action`可以处理异步任务，而`Mutation`必须只能同步。<br>
在异步流程中，先异步获得所需的数据，然后将返回的数据组合成`Action`。<br>
在生成`Action`之前，有个`createAction函数`。
```js
//Redux 思维的Action
const createAction = async (url) => {
  const resData = await asyncGetData(url)
  
  return {
    type: 'SOME_TYPE',
    resData  
  };
};

//Action 是对象
const Action = createAction('some url')

store.commit(Action)
```

在`Vuex`中的`Action`和`Redux`是有区别的<br>
在`Vuex`中的`Action`充当着`createAction`的功能<br>

`commit`应当只在`action`中使用，在`commit`之前还有`dispatch`<br>

在`Redux`中，`dispatch`分发的直接是`action`<br>
在`Vuex`中，`dispatch`分发的是`createAction或者mutation`，之后再`commit action`<br>

**Action 不同于 Mutation**
- Action提交的是mutation,而不是直接变更状态
- Action可以包含任意异步操作

```js
const store = new Vuex.Store({
  state: {
    count: 0
  },
  mutations: {
    increment (state) {
      state.count++
    }
  },
  actions: {
    increment (context) {
      context.commit('increment')
    }
  }
})
```
Action函数接受一个与`store实例`具有相同方法和属性的`context对象`<br>
其中`context`不是`store实例`

**分发Action**
```js
store.dispatch('increment')
```

**组件中分发Action**
组件中使用`this.$store.dispatch('xxx')`<br>
或者使用`mapActions`辅助函数将`actions`映射到`methods`中

**组合Action**
`dispatch`函数在结束后返回一个`Promise`
```js
actions: {
  actionA ({ commit }) {
    return new Promise((resolve, reject) => {
      setTimeout(() => {
        commit('someMutation')
        resolve()
      }, 1000)
    })
  }
}
```

##Module
可以将`store`分割成`模块Module`<br>
每个模块拥有自己的State、Mutation、Action、Getter<br>
然后将所有模块组合成一个，就形成了一个状态树。
```js
//一般我们可以按页面或者功能模块来划分模块
//大型项目可以按一个大模块划分
//然后在大漠块中再按页面划分
//如果还要更细，页面也可以切割成多个模块

//页面A的状态
const pageA = {
  state: { ... },
  mutations: { ... },
  actions: { ... },
  getters: { ... }
}

//页面B的状态
const pageAB = {
  state: { ... },
  mutations: { ... },
  actions: { ... }
}

const store = new Vuex.Store({
  modules: {
    a: pageA,
    b: pageAB
  }
})

store.state.a // -> pageA 的状态
store.state.b // -> pageAB 的状态
```

**模块的局部状态**

每个模块的`state`都是局部状态，模块中的`getter`、`mutation`、传进来的`state`都是局部的<br>
而`action`可以通过`context.state`拿到局部的状态,`context.rootState`拿到全局的<br>
不同于`mutation`，`getter`也可以拿到全局状态，`getter的第三个参数rootState`

**命名空间**

默认情况下，模块内部的action、mutation和getter是注册在全局命名空间的--多个模块对同一mutation或action作出响应<br>
`多个模块对同一mutation或action作出响应`，类似一个事件拥有多个处理程序（观察者模式）<br>
在模块中添加`namespace: true`可以避免添加到全局队列中

添加命名空间后的`getter : (state, getters, rootState, rootGetters) => {}`<br>
添加命名空间后的`action context : getters访问局部的 rootGetters访问全局的`<br>

如果添加了命名空间，但是还是想暴露某个`action`或`getter`为全局，使用`root:true`
```js
{
  actions: {
    someOtherAction ({dispatch}) {
      dispatch('someAction')
    }
  },
  modules: {
    foo: {
      namespaced: true,

      actions: {
        someAction: {
          root: true,
          handler (namespacedContext, payload) { ... } // -> 'someAction'
        }
      }
    }
  }
}
```

`mapState`、`mapGetters`、`mapActions`、`mapMutations`绑定带命名空间的模块。<br>

```js
computed: {
  ...mapState('some/nested/module', {
    a: state => state.a,
    b: state => state.b
  })
},
methods: {
  ...mapActions('some/nested/module', [
    'foo',
    'bar'
  ])
}
```

还可以使用`createNamespacedHelpers`来绑定命名空间值，类似`bind(context)`<br>
```js
import { createNamespacedHelpers } from 'vuex';

const { mapState, mapActions } = createNamespacedHelpers('some/nested/module');

export default {
  computed: {
    // 在 `some/nested/module` 中查找
    ...mapState({
      a: state => state.a,
      b: state => state.b
    })
  },
  methods: {
    // 在 `some/nested/module` 中查找
    ...mapActions([
      'foo',
      'bar'
    ])
  }
}
```

**模块动态注册**
```js
// 注册模块 `myModule`
store.registerModule('myModule', {
  // ...
})
// 注册嵌套模块 `nested/myModule`
store.registerModule(['nested', 'myModule'], {
  // ...
})

//卸载'myModule'，只能卸载动态装载的模块
 store.unregisterModule('myModule')
```

**应用**

如果是小型应用，可以按页面来分模块
```js
//pagea.js
export default {
  namespace:true,
  state: {},
  getters: {},
  mutations: {},
  actions: {}
}

//pageb.js
//pagec.js
//paged.js
//以上页面也是按pagea.js的方式
```

然后用一个文件引进所有模块，且全部暴露`export`
```js
//modules.js
import pageA from './pagea.js'
import pageB from './pageb.js'
import pageC from './pagec.js'
import pageD from './paged.js'

export default {
  pageA,
  pageB,
  pageC,
  pageD
}

```

最后在`main.js`引入
```js
//main.js
import modules from './modules/modules'

//将模块装进Store树中
const store = new Vuex.Store({
  //全局
  state: {},
  getters: {},
  mutations: {},
  actions: {},
  //模块
  modules
}) 
```

```js
//在组件中可以
this.$store.state.pageA
this.$store.state.pageB
this.$store.state.pageC
this.$store.state.pageD
```

