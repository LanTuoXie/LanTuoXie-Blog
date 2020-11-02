# 面试中遇到有关Vue的问题

## 1、兄弟之间传值的几种方式

最常用的`VueX`，或者本地存储`localStorage`、`sessionStorage`、`cookie`，或者全局`window`添加、或者通过`Vue.prototype`原型链的方式，还有`Vue.observable()`函数的方式，以及`provide/inject`

`Vue.observable()`的方式可以将一个共享的数据独立成一个独立的文件。

```js
import Vue from 'vue'

// 使用的时候直接将store map 给 computed
export const store = Vue.observable({ count: 0 })

// 使用的时候直接将mutations 传给 actions
export const mutations = {
  setCount(count) {
    store.count = count
  },
  addCount() {
    store.count++
  },
  delCount() {
    store.count--
  }
}
```

使用到组件中:

```html
<script>
  import { store, mutations } from '@/store/oneStore.js'

  export default {
    data() {
      return {}
    },
    computed: {
      count() {
        return store.count
      }
    },
    methods: { ...mutations }
  }
</script>
```

`eventBus` 的方式 `Vue.$on` 和 `Vue.$emit`，甚至可以自己定义一个，本质就是 `订阅与发布模式`，这里还可以 `Vue.$off/Vue.$once` 如果是一次性的状态，好处还是独立出一个独立的文件 `eventBus.js`。

```js
import Vue from 'vue'

export default const eventBus = new Vue()
```

总结：

- `VueX`
- `localStorage`、`sessionStorage`、`cookie`
- `全局window`
- `Vue.prototype`
- `Vue.observable`
- `eventBus/订阅与发布模式`

## 2、父子之间传值

- `props`的方式，`单向数据流`
- `$emit`子组件传给父组件
- `状态提升`：父组件统一管理状态，父组件暴露可以修改状态的`动作Action`给子组件调用。

## 3、scope-slot的实现原理
