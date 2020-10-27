# VueX源码分析(2)

剩余内容

- `/module`
- `/plugins`
- `helpers.js`
- `store.js`

helpers要从底部开始分析比较好。也即先从辅助函数开始再分析那4个map函数`mapState`。

## helpers.js

**getModuleByNamespace:**

```js
/**
 * Search a special module from store by namespace. if module not exist, print error message.
 * @param {Object} store
 * @param {String} helper
 * @param {String} namespace
 * @return {Object}
 */
function getModuleByNamespace (store, helper, namespace) {
  const module = store._modulesNamespaceMap[namespace]
  if (process.env.NODE_ENV !== 'production' && !module) {
    console.error(`[vuex] module namespace not found in ${helper}(): ${namespace}`)
  }
  return module
}
```

解析：

- 通过namespace来寻找module，如果找不到打印错误信息（开发环境）
- `_modulesNamespaceMap`这个Map存有所有的module
- 在vuex中，不同的作用域用'/'来分隔开的(嵌套模块)，如商城中的购物车的namespace可以这样表示'shop/shopping_cart'

**normalizeMap:**

```js
/**
 * Normalize the map
 * normalizeMap([1, 2, 3]) => [ { key: 1, val: 1 }, { key: 2, val: 2 }, { key: 3, val: 3 } ]
 * normalizeMap({a: 1, b: 2, c: 3}) => [ { key: 'a', val: 1 }, { key: 'b', val: 2 }, { key: 'c', val: 3 } ]
 * @param {Array|Object} map
 * @return {Object}
 */
function normalizeMap (map) {
  return Array.isArray(map)
    ? map.map(key => ({ key, val: key }))
    : Object.keys(map).map(key => ({ key, val: map[key] }))
}
```

解析：

- 将数组或者对象转化成[Map, Map]的格式，Map关键字有{ key, val }
- 如果是数组，生成Map的key === val
- 如果是对象，生成Map的key就是对象的键名，val就是对象的值

**normalizeNamespace:**

```js
/**
 * Return a function expect two param contains namespace and map. it will normalize the namespace and then the param's function will handle the new namespace and the map.
 * @param {Function} fn
 * @return {Function}
 */
function normalizeNamespace (fn) {
  return (namespace, map) => {
    if (typeof namespace !== 'string') {
      map = namespace
      namespace = ''
    } else if (namespace.charAt(namespace.length - 1) !== '/') {
      namespace += '/'
    }
    return fn(namespace, map)
  }
}
```

解析：

- 这里的fn就是mapState等4大map函数，使用柯里化缓存fn
- `typeof namespace !== 'string'`第一个判断是支持两种传参模式：1、可以不传namespace直接传map，如mapActions(['action'])；2、支持传namespace，如mapActions('shop', ['action'])
- 也即namespace可传可不传，不传最后初始化`namespace = ''`
- 如果传了namespace，要检查最后一个字符带不带`'/'`，没有则补全
- 这个函数就是在执行mapState、mapAction等4大map函数之前的namespace预处理，最终才把namesapce和map传个fn函数

**createNamespacedHelpers:**

```js
/**
 * Rebinding namespace param for mapXXX function in special scoped, and return them by simple object
 * @param {String} namespace
 * @return {Object}
 */
export const createNamespacedHelpers = (namespace) => ({
  mapState: mapState.bind(null, namespace),
  mapGetters: mapGetters.bind(null, namespace),
  mapMutations: mapMutations.bind(null, namespace),
  mapActions: mapActions.bind(null, namespace)
})
```

解析：

- 这个bind函数涉及到柯里化，要理解柯里化才可理解这个意思
- 柯里化和函数的参数个数有关，可以简单把柯里化理解成是一个收集参数的过程，只有收集够函数所需的参数个数，才会执行函数体，否则返回一个缓存了之前收集的参数的函数。
- 4大map函数都要接受两个参数，namespace和map
- 由柯里化：mapState函数有2个参数，要收集够2个参数才会执行mapState的函数体
- `createNamespacedHelpers`的作用是让mapState收集第一个参数namespace，由于还差一个参数map，所以返回的是一个缓存了namespace参数的函数，继续接收下一个参数map
- 所以被`createNamespacedHelpers`返回的mapState只需传入1个参数map就可以执行了，且传入的第一个参数必须是map，因为namespace已经收集到了，再传入namespace最终执行的结果会是mapState(namespace, namespace)
- 总之，如果了解过柯里化，这里应该很好理解。

## mapState、mapMutations、mapActions、mapGetters

**mapState:**

```js
/**
 * Reduce the code which written in Vue.js for getting the state.
 * @param {String} [namespace] - Module's namespace
 * @param {Object|Array} states # Object's item can be a function which accept state and getters for param, you can do something for state and getters in it.
 * @param {Object}
 */
export const mapState = normalizeNamespace((namespace, states) => {
  const res = {}
  normalizeMap(states).forEach(({ key, val }) => {
    res[key] = function mappedState () {
      let state = this.$store.state
      let getters = this.$store.getters
      if (namespace) {
        const module = getModuleByNamespace(this.$store, 'mapState', namespace)
        if (!module) {
          return
        }
        state = module.context.state
        getters = module.context.getters
      }
      return typeof val === 'function'
        ? val.call(this, state, getters)
        : state[val]
    }
    // mark vuex getter for devtools
    res[key].vuex = true
  })
  return res
})
```

解析：

- 4大map函数最终结果都是返回一个对象{}
- `mappedState`其实就是`computed`的属性的函数，看这个函数要联想到`computed`，且这个函数的`this`也是指向vue的
- 上面的`this.$state.state`和`this.$state.getters`是全局的`state`和`getters`
- 接下来就是判断是不是模块，是则拿到模块的state和getter。有种情况用到`mapState({ name: (state, getter) => state.name })`
- 最后返回 `val` 。如果是函数，如上面那样要先执行一遍，再返回函数执行后的值
- 因为mappedState就是computed中属性的函数，一定是要返回值的。
- `res`是个对象，所以可以`{ computed: { ...mapState(['name', 'age']) } }`

```js
// mapState(['name', 'age']) 
const res = {
  // { key, val } 其中： key = 'name' val = 'name'

  name: function mappedState () {
    // 没有命名空间的情况
    // 这个函数要用到computed的，这里this指向Vue组件实例
    return this.$store.state[name]
  },
  age: function mappedState () {
    // 如果有命名空间的情况
    // 如上面源码根据namespace拿到模块module
    const state = module.context.state
    return state[age]
  }
}

// mapState({ name: (state, getter) => state.name })
const res = {
  // { key, val } 其中：key = 'name' val = (state, getter) => state.name

  name: function mappedState () {
    // 没有命名空间
    // 如上面代码一样{ key, val }中的 val = (state, getter) => state.name }
    const state = this.$store.state
    cosnt getter = this.$store.getter
    // this 是指向Vue组件实例的
    return val.call(this, state, getter)
  }
}

```

**mapMutations:**

```js
/**
 * Reduce the code which written in Vue.js for committing the mutation
 * @param {String} [namespace] - Module's namespace
 * @param {Object|Array} mutations # Object's item can be a function which accept `commit` function as the first param, it can accept anthor params. You can commit mutation and do any other things in this function. specially, You need to pass anthor params from the mapped function.
 * @return {Object}
 */
export const mapMutations = normalizeNamespace((namespace, mutations) => {
  const res = {}
  normalizeMap(mutations).forEach(({ key, val }) => {
    res[key] = function mappedMutation (...args) {
      // Get the commit method from store
      let commit = this.$store.commit
      if (namespace) {
        const module = getModuleByNamespace(this.$store, 'mapMutations', namespace)
        if (!module) {
          return
        }
        commit = module.context.commit
      }
      return typeof val === 'function'
        ? val.apply(this, [commit].concat(args))
        : commit.apply(this.$store, [val].concat(args))
    }
  })
  return res
})
```

解析：

- 这里也要判断是不是模块，不同情况的commit不同，是用全局的还是用模块的
- `mappedMutation`是`methods`的函数，this同样指向Vue的实例
- `val.apply(this, [commit].concat(args))`，是这种情况`mapMutations({ mutationName: (commit, ...arg) => commit('自定义') })`
- `commit.apply(this.$store, [val].concat(args))`，是这种情况`mapMutations(['CHANGE_NAME'])`使用的时候还可以传参数`this['CHANGE_NAME'](name)`

**mapGetters:**

```js
/**
 * Reduce the code which written in Vue.js for getting the getters
 * @param {String} [namespace] - Module's namespace
 * @param {Object|Array} getters
 * @return {Object}
 */
export const mapGetters = normalizeNamespace((namespace, getters) => {
  const res = {}
  normalizeMap(getters).forEach(({ key, val }) => {
    // thie namespace has been mutate by normalizeNamespace
    val = namespace + val
    res[key] = function mappedGetter () {
      if (namespace && !getModuleByNamespace(this.$store, 'mapGetters', namespace)) {
        return
      }
      if (process.env.NODE_ENV !== 'production' && !(val in this.$store.getters)) {
        console.error(`[vuex] unknown getter: ${val}`)
        return
      }
      return this.$store.getters[val]
    }
    // mark vuex getter for devtools
    res[key].vuex = true
  })
  return res
})
```

解析：

- `val = namespace + val`这里是，不管是模块的getter还是全局的getter最终都存在一个地方中($store.getters)，是模块的会有'/，所以这里要补充`namespace + val`
- 所以最后返回的是`this.$store.getters[val]`
- 还有`mappedGetter`对应`computed`属性的函数，this指向Vue实例

**mapActions:**

```js
/**
 * Reduce the code which written in Vue.js for dispatch the action
 * @param {String} [namespace] - Module's namespace
 * @param {Object|Array} actions # Object's item can be a function which accept `dispatch` function as the first param, it can accept anthor params. You can dispatch action and do any other things in this function. specially, You need to pass anthor params from the mapped function.
 * @return {Object}
 */
export const mapActions = normalizeNamespace((namespace, actions) => {
  const res = {}
  normalizeMap(actions).forEach(({ key, val }) => {
    res[key] = function mappedAction (...args) {
      // get dispatch function from store
      let dispatch = this.$store.dispatch
      if (namespace) {
        const module = getModuleByNamespace(this.$store, 'mapActions', namespace)
        if (!module) {
          return
        }
        dispatch = module.context.dispatch
      }
      return typeof val === 'function'
        ? val.apply(this, [dispatch].concat(args))
        : dispatch.apply(this.$store, [val].concat(args))
    }
  })
  return res
})
```

解析：

- 这个和`mapMutations`差不多，只是`commit`换成了`dispatch`
- `mappedAction`对应`methods`的属性的函数，`this`也是指向Vue实例
