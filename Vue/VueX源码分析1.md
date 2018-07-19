##VueX源码分析（1）

文件架构如下

- `/module`
- `/plugins`
- `helpers.js`
- `index.esm.js`
- `index.js`
- `store.js`
- `util.js`

#### util.js

先从最简单的工具函数开始。

**find函数**

```js
/**
 * Get the first item that pass the test
 * by second argument function
 *
 * @param {Array} list
 * @param {Function} f
 * @return {*}
 */
export function find (list, f) {
  return list.filter(f)[0]
}
```

find函数的测试用例

```js
it('find', () => {
  const list = [33, 22, 112, 222, 43]
  expect(find(list, function (a) { return a % 2 === 0 })).toEqual(22)
})
```

解析：   
- 先用`断言函数f`过滤`列表list`，最后取过滤后列表的第一个元素。

**deepCopy函数**

```js
/**
 * Deep copy the given object considering circular structure.
 * This function caches all nested objects and its copies.
 * If it detects circular structure, use cached copy to avoid infinite loop.
 *
 * @param {*} obj
 * @param {Array<Object>} cache
 * @return {*}
 */
export function deepCopy (obj, cache = []) {
  // just return if obj is immutable value
  if (obj === null || typeof obj !== 'object') {
    return obj
  }

  // if obj is hit, it is in circular structure
  const hit = find(cache, c => c.original === obj)
  if (hit) {
    return hit.copy
  }

  const copy = Array.isArray(obj) ? [] : {}
  // put the copy into cache at first
  // because we want to refer it in recursive deepCopy
  cache.push({
    original: obj,
    copy
  })

  Object.keys(obj).forEach(key => {
    copy[key] = deepCopy(obj[key], cache)
  })

  return copy
}
```

deepCopy的测试用例

```js
  // 普通结构
  it('deepCopy: nornal structure', () => {
    const original = {
      a: 1,
      b: 'string',
      c: true,
      d: null,
      e: undefined
    }
    const copy = deepCopy(original)

    expect(copy).toEqual(original)
  })
  
  // 嵌套结构
  it('deepCopy: nested structure', () => {
    const original = {
      a: {
        b: 1,
        c: [2, 3, {
          d: 4
        }]
      }
    }
    const copy = deepCopy(original)

    expect(copy).toEqual(original)
  })
  
  // 循环引用结构
  it('deepCopy: circular structure', () => {
    const original = {
      a: 1
    }
    original.circular = original

    const copy = deepCopy(original)

    expect(copy).toEqual(original)
  })
```

解析：  
- 功能：支持循环引用的深克隆函数
- 第一个if判断`obj === null || typeof obj !== 'object'`判断如果不是引用类型直接返回(基本类型是值拷贝)，也是递归的一个出口。   
- 第二个判断`hit`是判断是不是循环引用，由于是循环引用，在cache中应该有缓存到一份拷贝，直接取cache的，避免再次重复拷贝一份。
- 什么是循环引用看测试用例第三个`original.circular = original`，循环引用和被引用的内容是一样的，用缓存就是避免重复的克隆（内容一样）
- `original.circular`是循环引用，`original`是被循环引用
- 先把`cope`放到`cache`中，是在递归的时候，如果遇到循环引用，要确保cache中有一份`被循环引用的copy`，但是`copy`必须是引用类型。
- 为什么`cope`必须是引用类型？`循环引用`保存的是引用不是内容（这时候还没拷贝完），在递归的时候并未完成拷贝，只有递归跑完了才完成拷贝，这样未来`被循环引用`的内容改变时（拷贝完），`循环引用`的内容同步改变
- 所以`const copy = Array.isArray(obj) ? [] : {}`必须是引用类型。
- 最后`Object.keys`可以遍历对象和数组的所有键名（只返回实例的属性，不包含原型链和Symbol），实现递归克隆。
- 一共两个出口，一个是基本类型，另一个是循环引用。

**forEachValue**

```js
/**
 * forEach for object
 */
export function forEachValue (obj, fn) {
  Object.keys(obj).forEach(key => fn(obj[key], key))
}
```

测试用例
```js
  it('forEachValue', () => {
    let number = 1

    function plus (value, key) {
      number += value
    }
    const origin = {
      a: 1,
      b: 3
    }

    forEachValue(origin, plus)
    expect(number).toEqual(5)
  })
```

解析：   
- 一个遍历对象的函数（支持对象和数组）
- `fn(value, key)`但是回调函数第一个参数是值，第二个参数是键值

**isObject**

```js
export function isObject (obj) {
  return obj !== null && typeof obj === 'object'
}
```

测试用例
```js
  it('isObject', () => {
    expect(isObject(1)).toBe(false)
    expect(isObject('String')).toBe(false)
    expect(isObject(undefined)).toBe(false)
    expect(isObject({})).toBe(true)
    expect(isObject(null)).toBe(false)
    expect(isObject([])).toBe(true)
    expect(isObject(new Function())).toBe(false)
  })
```

解析：   
- 判断是不是对象，这里没有判断是不是原生对象，数组也是通过的。
- 由于typeof null === 'object'要先判断是不是null

**isPromise**

```js
export function isPromise (val) {
  return val && typeof val.then === 'function'
}
```

测试用例
```js
  it('isPromise', () => {
    const promise = new Promise(() => {}, () => {})
    expect(isPromise(1)).toBe(false)
    expect(isPromise(promise)).toBe(true)
    expect(isPromise(new Function())).toBe(false)
  })
```

解析：   
- 判断是不是Promise
- 首先判断val不是undefined，然后才可以判断val.then，避免报错
- 判断依据是val.then是不是函数

**assert**

```js
export function assert (condition, msg) {
  if (!condition) throw new Error(`[vuex] ${msg}`)
}
```

测试用例：
```js
  it('assert', () => {
    expect(assert.bind(null, false, 'Hello')).toThrowError('[vuex] Hello')
  })
```

解析：   
- 断言函数，断言不通过抛出一个自定义错误信息的Error

#### `index.js`和`index.esm.js`

`index.js`
```js
import { Store, install } from './store'
import { mapState, mapMutations, mapGetters, mapActions, createNamespacedHelpers } from './helpers'

export default {
  Store,
  install,
  version: '__VERSION__',
  mapState,
  mapMutations,
  mapGetters,
  mapActions,
  createNamespacedHelpers
}

```


`index.esm.js`
```js
import { Store, install } from './store'
import { mapState, mapMutations, mapGetters, mapActions, createNamespacedHelpers } from './helpers'

export default {
  Store,
  install,
  version: '__VERSION__',
  mapState,
  mapMutations,
  mapGetters,
  mapActions,
  createNamespacedHelpers
}

export {
  Store,
  install,
  mapState,
  mapMutations,
  mapGetters,
  mapActions,
  createNamespacedHelpers
}

```

解析：   
- 区别就是`index.esm.js`比`index.js`多了个一个导入模式
- `import Vuex, { mapState } from 'index.esm.js'`：有两种方式导入
- `import Vuex from 'index.js'`：只有一种方式导入

#### mixin.js

```js
export default function (Vue) {
  const version = Number(Vue.version.split('.')[0])

  if (version >= 2) {
    Vue.mixin({ beforeCreate: vuexInit })
  } else {
    // override init and inject vuex init procedure
    // for 1.x backwards compatibility.
    const _init = Vue.prototype._init
    Vue.prototype._init = function (options = {}) {
      options.init = options.init
        ? [vuexInit].concat(options.init)
        : vuexInit
      _init.call(this, options)
    }
  }

  /**
   * Vuex init hook, injected into each instances init hooks list.
   */

  function vuexInit () {
    const options = this.$options
    // store injection
    if (options.store) {
      this.$store = typeof options.store === 'function'
        ? options.store()
        : options.store
    } else if (options.parent && options.parent.$store) {
      this.$store = options.parent.$store
    }
  }
}

```

解析：   
- 为什么每个组件都拥有$store属性，也即每个组件都能拿到$store
- Vue2直接用mixin和钩子函数beforeCreate，Vue1用外观（装饰者）模式重写Vue._init函数。
- `vuexInit`是将全局注册的store注入到当前组件中，在创建该组件之前
- $options是`new Vue(options)`的options,$options中有store
- 由于`beforeCreate`是`Vue`的周期钩子，`this`指向当前组件实例，所以`this.$store`可以把store直接注入当前组件
- 所有组件都是继承于一个全局Vue的，全局mixin组件周期钩子`beforeCreate`，这样每个组件都能自动注入store，也即每个组件都能直接通过`$store`拿到全局Vue`new Vue({ el: 'app', store, router })`的`store`
