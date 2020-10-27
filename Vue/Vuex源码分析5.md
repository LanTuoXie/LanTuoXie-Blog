# VueX源码分析(5)

最终也是最重要的`store.js`，该文件主要涉及的内容如下：

- Store类
- genericSubscribe函数
- resetStore函数
- resetStoreVM函数
- installModule函数
- makeLocalContext函数
- makeLocalGetters函数
- registerMutation函数
- registerAction函数
- registerGetter函数
- enableStrictMode函数
- getNestedState函数
- unifyObjectStyle函数
- install函数
- `let Vue`上面这些和`这个Vue`都在同一个作用域

**install:**

```js
export function install (_Vue) {
  if (Vue && _Vue === Vue) {
    if (process.env.NODE_ENV !== 'production') {
      console.error(
        '[vuex] already installed. Vue.use(Vuex) should be called only once.'
      )
    }
    return
  }
  Vue = _Vue
  applyMixin(Vue)
}
```

解析：

- 主要是让所有组件都能拿到store,在组件生命周期`beforeCreate`期间给所有组件创建`$store`
- 如果是开发环境还会判断是否使用`Vue.use(Vuex)`

**unifyObjectStyle:**

```js
function unifyObjectStyle (type, payload, options) {
  if (isObject(type) && type.type) {
    options = payload
    payload = type
    type = type.type
  }

  if (process.env.NODE_ENV !== 'production') {
    assert(typeof type === 'string', `expects string as the type, but found ${typeof type}.`)
  }

  return { type, payload, options }
}
```

解析：

- 主要处理`dispatch('pushTab', payload, options)` 和 `dispatch({ type: 'pushTab', payload }, options)`这种情况
- 也即第一个参数可以是字符串，也可以是对象，就是由第一个参数决定采用什么样的方式传参。这个函数的目的就是支持不同风格的传参
- 传参风格（方式），由传入的第一个参数类型决定

**getNestedState:**

```js
function getNestedState (state, path) {
  return path.length
    ? path.reduce((state, key) => state[key], state)
    : state
}
```

解析：

- 和之前模块module中的path一样，path为空数组`[]`为非嵌套或根模块，`{ shop: { card: { item: 1 } } }`的path为`['shop', 'card', 'item']`
- 就是通过迭代的方式获取状态`item`
- 这样做的好处是：对象和数组都支持，如果是数组`[{ name: 'getMe' }, { name: 'notMe' }]`，要获取`'getMe'`那么`const path = [0, 'name']`

**enableStrictMode:**

```js
function enableStrictMode (store) {
  store._vm.$watch(function () { return this._data.$$state }, () => {
    if (process.env.NODE_ENV !== 'production') {
      assert(store._committing, `do not mutate vuex store state outside mutation handlers.`)
    }
  }, { deep: true, sync: true })
}
```

解析：

- `$watch`就是`Vue.$watch`，主要功能是：如果启动了严格模式，监控数据状态的改变是不是通过`commit`改变的。
- 如果不是通过`commit`改变状态的，在开发模式下会提示。
- `VueX`的状态是存在于一个`Vue实例中的_data.$$state`，`Store._vm`可以解释`VueX`为什么叫`VueX`
- 可以说`VueX`是`Vue的一个实例`

**genericSubscribe:**

```js
function genericSubscribe (fn, subs) {
  if (subs.indexOf(fn) < 0) {
    subs.push(fn)
  }
  return () => {
    const i = subs.indexOf(fn)
    if (i > -1) {
      subs.splice(i, 1)
    }
  }
}
```

解析：

- 订阅某个观察者，这里面`subs`是观察者，`subs`关联到某个状态，那个状态改变，会遍历`subs`调用里面的函数。
- 返回的函数时可以取消订阅的，也即`const unsubscribe = genericSubscribe(fn, subs)`，调用`unsubscribe()`就可以取消订阅

## class Store

几个辅助函数和状态相关

- resetStore：重置状态，类似重新开始游戏，会重新创建Store
- resetStoreVM：Vuex的状态是存放在这个Vue实例的_data中
- installModule：安装模块，注册模块
- makeLocalContext：作用域模块中定义的getters,actions等的函数都会传入一个`context`
- makeLocalGetters：作用域模块中定义getters细节
- registerMutation：模块中的Mutation注册
- registerAction：模块中的Action注册
- registerGetter：模块中的Getter注册

```js
let Vue // bind on install

export class Store {
  constructor (options = {}) {
    // Auto install if it is not done yet and `window` has `Vue`.
    // To allow users to avoid auto-installation in some cases,
    // this code should be placed here. See #731
    if (!Vue && typeof window !== 'undefined' && window.Vue) {
      install(window.Vue)
    }

    if (process.env.NODE_ENV !== 'production') {
      assert(Vue, `must call Vue.use(Vuex) before creating a store instance.`)
      assert(typeof Promise !== 'undefined', `vuex requires a Promise polyfill in this browser.`)
      assert(this instanceof Store, `store must be called with the new operator.`)
    }

    const {
      plugins = [],
      strict = false
    } = options

    // store internal state
    this._committing = false
    this._actions = Object.create(null)
    this._actionSubscribers = []
    this._mutations = Object.create(null)
    this._wrappedGetters = Object.create(null)
    this._modules = new ModuleCollection(options)
    this._modulesNamespaceMap = Object.create(null)
    this._subscribers = []
    this._watcherVM = new Vue()

    // bind commit and dispatch to self
    const store = this
    const { dispatch, commit } = this
    this.dispatch = function boundDispatch (type, payload) {
      return dispatch.call(store, type, payload)
    }
    this.commit = function boundCommit (type, payload, options) {
      return commit.call(store, type, payload, options)
    }

    // strict mode
    this.strict = strict

    const state = this._modules.root.state

    // init root module.
    // this also recursively registers all sub-modules
    // and collects all module getters inside this._wrappedGetters
    installModule(this, state, [], this._modules.root)

    // initialize the store vm, which is responsible for the reactivity
    // (also registers _wrappedGetters as computed properties)
    resetStoreVM(this, state)

    // apply plugins
    plugins.forEach(plugin => plugin(this))

    if (Vue.config.devtools) {
      devtoolPlugin(this)
    }
  }

  get state () {
    return this._vm._data.$$state
  }

  set state (v) {
    if (process.env.NODE_ENV !== 'production') {
      assert(false, `use store.replaceState() to explicit replace store state.`)
    }
  }

  commit (_type, _payload, _options) {
    // check object-style commit
    const {
      type,
      payload,
      options
    } = unifyObjectStyle(_type, _payload, _options)

    const mutation = { type, payload }
    const entry = this._mutations[type]
    if (!entry) {
      if (process.env.NODE_ENV !== 'production') {
        console.error(`[vuex] unknown mutation type: ${type}`)
      }
      return
    }
    this._withCommit(() => {
      entry.forEach(function commitIterator (handler) {
        handler(payload)
      })
    })
    this._subscribers.forEach(sub => sub(mutation, this.state))

    if (
      process.env.NODE_ENV !== 'production' &&
      options && options.silent
    ) {
      console.warn(
        `[vuex] mutation type: ${type}. Silent option has been removed. ` +
        'Use the filter functionality in the vue-devtools'
      )
    }
  }

  dispatch (_type, _payload) {
    // check object-style dispatch
    const {
      type,
      payload
    } = unifyObjectStyle(_type, _payload)

    const action = { type, payload }
    const entry = this._actions[type]
    if (!entry) {
      if (process.env.NODE_ENV !== 'production') {
        console.error(`[vuex] unknown action type: ${type}`)
      }
      return
    }

    this._actionSubscribers.forEach(sub => sub(action, this.state))

    return entry.length > 1
      ? Promise.all(entry.map(handler => handler(payload)))
      : entry[0](payload)
  }

  subscribe (fn) {
    return genericSubscribe(fn, this._subscribers)
  }

  subscribeAction (fn) {
    return genericSubscribe(fn, this._actionSubscribers)
  }

  watch (getter, cb, options) {
    if (process.env.NODE_ENV !== 'production') {
      assert(typeof getter === 'function', `store.watch only accepts a function.`)
    }
    return this._watcherVM.$watch(() => getter(this.state, this.getters), cb, options)
  }

  replaceState (state) {
    this._withCommit(() => {
      this._vm._data.$$state = state
    })
  }

  registerModule (path, rawModule, options = {}) {
    if (typeof path === 'string') path = [path]

    if (process.env.NODE_ENV !== 'production') {
      assert(Array.isArray(path), `module path must be a string or an Array.`)
      assert(path.length > 0, 'cannot register the root module by using registerModule.')
    }

    this._modules.register(path, rawModule)
    installModule(this, this.state, path, this._modules.get(path), options.preserveState)
    // reset store to update getters...
    resetStoreVM(this, this.state)
  }

  unregisterModule (path) {
    if (typeof path === 'string') path = [path]

    if (process.env.NODE_ENV !== 'production') {
      assert(Array.isArray(path), `module path must be a string or an Array.`)
    }

    this._modules.unregister(path)
    this._withCommit(() => {
      const parentState = getNestedState(this.state, path.slice(0, -1))
      Vue.delete(parentState, path[path.length - 1])
    })
    resetStore(this)
  }

  hotUpdate (newOptions) {
    this._modules.update(newOptions)
    resetStore(this, true)
  }

  _withCommit (fn) {
    const committing = this._committing
    this._committing = true
    fn()
    this._committing = committing
  }
}
```

解析：

- `install(window.Vue)`如果是Vue2，通过mixins的方式添加`beforeCreate`钩子函数，把store传给任何继承这个Vue的实例（组件），所以所有组件都拥有一个$store的属性。也即每个组件都能拿到store。

- 第二个断言判断是确保Vue在当前环境中，且需要Promise，强制Store作为构造函数。

- 可配置项有两个值`plugins`和是否使用`strict`严格模式（只能通过commit改变状态），`plugins`一般用于开发调试

- 将全局（非模块内）的`dispatch和commit`的`this`绑定到`store`

Store类的静态属性

- `_committing`：记录当前是否commit中
- `_actions`：记录所有action的字段，有模块作用域的用'/'分隔
- `_actionSubscribers`：全局的dispatch后遍历调用数组中的函数
- `_mutations`：和actions一样
- `_wrappedGetters`：不管是使用了模块还是全局的getter都存在于此
- `_modules`:全局模块
- `_modulesNamespaceMap`：所有模块全存于此，可通过namespace取出想要的模块
- `_subscribers`：全局的commit调用之后，遍历调用数组中的函数
- `_watcherVM`：Vue实例用于`watch()`函数
- `strict`：是否使用严格模式，使用那么改变状态只能通过commit来改变状态
- `state`：Store的所有state包含模块的
- `_vm`：Vue的实例，`VueX`真正状态是存于这里`Store._vm._data.$$state`

**installModule:**

```js
function installModule (store, rootState, path, module, hot) {
  const isRoot = !path.length
  const namespace = store._modules.getNamespace(path)

  // register in namespace map
  if (module.namespaced) {
    store._modulesNamespaceMap[namespace] = module
  }

  // set state
  if (!isRoot && !hot) {
    const parentState = getNestedState(rootState, path.slice(0, -1))
    const moduleName = path[path.length - 1]
    store._withCommit(() => {
      Vue.set(parentState, moduleName, module.state)
    })
  }

  const local = module.context = makeLocalContext(store, namespace, path)

  module.forEachMutation((mutation, key) => {
    const namespacedType = namespace + key
    registerMutation(store, namespacedType, mutation, local)
  })

  module.forEachAction((action, key) => {
    const type = action.root ? key : namespace + key
    const handler = action.handler || action
    registerAction(store, type, handler, local)
  })

  module.forEachGetter((getter, key) => {
    const namespacedType = namespace + key
    registerGetter(store, namespacedType, getter, local)
  })

  module.forEachChild((child, key) => {
    installModule(store, rootState, path.concat(key), child, hot)
  })
}
```

- 这个函数主要作用是安装模块，在初始化根模块的同时注册所有子模块，以及将所有getter(包括模块中的)收集到`this._wrappedGetters`中。    
- `_modulesNamespaceMap`存放所有模块，可以通过namespaced来获取模块（数据结构：哈希表）  
- `Vue.set(parentState, moduleName, module.state)`由于`VueX`是Vue的实例，Vue设置的状态，它的实例(VueX)可以继承
- 创建每个模块的`context`

**makeLocalContext:**

```js
/**
 * make localized dispatch, commit, getters and state
 * if there is no namespace, just use root ones
 */
function makeLocalContext (store, namespace, path) {
  const noNamespace = namespace === ''

  const local = {
    dispatch: noNamespace ? store.dispatch : (_type, _payload, _options) => {
      const args = unifyObjectStyle(_type, _payload, _options)
      const { payload, options } = args
      let { type } = args

      if (!options || !options.root) {
        type = namespace + type
        if (process.env.NODE_ENV !== 'production' && !store._actions[type]) {
          console.error(`[vuex] unknown local action type: ${args.type}, global type: ${type}`)
          return
        }
      }

      return store.dispatch(type, payload)
    },

    commit: noNamespace ? store.commit : (_type, _payload, _options) => {
      const args = unifyObjectStyle(_type, _payload, _options)
      const { payload, options } = args
      let { type } = args

      if (!options || !options.root) {
        type = namespace + type
        if (process.env.NODE_ENV !== 'production' && !store._mutations[type]) {
          console.error(`[vuex] unknown local mutation type: ${args.type}, global type: ${type}`)
          return
        }
      }

      store.commit(type, payload, options)
    }
  }

  // getters and state object must be gotten lazily
  // because they will be changed by vm update
  Object.defineProperties(local, {
    getters: {
      get: noNamespace
        ? () => store.getters
        : () => makeLocalGetters(store, namespace)
    },
    state: {
      get: () => getNestedState(store.state, path)
    }
  })

  return local
}
```

- 这个函数主要创建局部的dispatch、commit、getters、state。存于`context`  
- 局部的意思是只取当前作用域模块的getters、state以及dispatch和commit，没有作用域就取全局的  
- get和state采用数据劫持和懒获取的方式
- 懒：就是一个函数`() => store.getters`，只有调用这个函数才获取到所有getters，结合`get`就是只有引用这个属性才会获取所有getters
- 懒获取就是把原本的操作封装成函数，在需要的时候调用该函数即可获得。实际上就是宏命令或者叫命令模式，用一个函数把一块要执行的命令封装起来。
- `() => fn()`要留意的是：是`fn()`而不是`fn`，`fn()`才是`要执行的命令`

**makeLocalGetters:**

```js
function makeLocalGetters (store, namespace) {
  const gettersProxy = {}

  const splitPos = namespace.length
  Object.keys(store.getters).forEach(type => {
    // skip if the target getter is not match this namespace
    if (type.slice(0, splitPos) !== namespace) return

    // extract local getter type
    const localType = type.slice(splitPos)

    // Add a port to the getters proxy.
    // Define as getter property because
    // we do not want to evaluate the getters in this time.
    Object.defineProperty(gettersProxy, localType, {
      get: () => store.getters[type],
      enumerable: true
    })
  })

  return gettersProxy
}
```

- 先判断有没有匹配的作用域（getter = namespace + getterName），然后取出getter的名称        
- 通过代理的方式返回这个getter

**resetStoreVM:**

```js
function resetStoreVM (store, state, hot) {
  const oldVm = store._vm

  // bind store public getters
  store.getters = {}
  const wrappedGetters = store._wrappedGetters
  const computed = {}
  forEachValue(wrappedGetters, (fn, key) => {
    // use computed to leverage its lazy-caching mechanism
    computed[key] = () => fn(store)
    Object.defineProperty(store.getters, key, {
      get: () => store._vm[key],
      enumerable: true // for local getters
    })
  })

  // use a Vue instance to store the state tree
  // suppress warnings just in case the user has added
  // some funky global mixins
  const silent = Vue.config.silent
  Vue.config.silent = true
  store._vm = new Vue({
    data: {
      $$state: state
    },
    computed
  })
  Vue.config.silent = silent

  // enable strict mode for new vm
  if (store.strict) {
    enableStrictMode(store)
  }

  if (oldVm) {
    if (hot) {
      // dispatch changes in all subscribed watchers
      // to force getter re-evaluation for hot reloading.
      store._withCommit(() => {
        oldVm._data.$$state = null
      })
    }
    Vue.nextTick(() => oldVm.$destroy())
  }
}
```

- 主要是重新设置`store._vm`的值    
- `_vm`的$$state保存的是store.state
- `_vm`的`computed`是`store._wrappedGetters`的值
- 可以看出`VueX`的状态是存在于一个`Vue实例`的$data中的
- （数据劫持+懒调用）和computed就是getter的实现原理，也是为什么getter有缓存的效果
- 这个computed是`VueX`内部的`_vm`的，也可以认为就是getters

**registerMutation:**

```js
function registerMutation (store, type, handler, local) {
  const entry = store._mutations[type] || (store._mutations[type] = [])
  entry.push(function wrappedMutationHandler (payload) {
    handler.call(store, local.state, payload)
  })
}
```

- 主要是按命名空间划分和收集不同命名空间的mutaion，以及把它们的this绑定为store
- local.state如果有命名空间使用命名空间的，没有使用全局的
- `store._mutations[type]`，type就是命名空间，按命名空间来存放mutations

**registerAction:**

```js
function registerAction (store, type, handler, local) {
  const entry = store._actions[type] || (store._actions[type] = [])
  entry.push(function wrappedActionHandler (payload, cb) {
    let res = handler.call(store, {
      dispatch: local.dispatch,
      commit: local.commit,
      getters: local.getters,
      state: local.state,
      rootGetters: store.getters,
      rootState: store.state
    }, payload, cb)
    if (!isPromise(res)) {
      res = Promise.resolve(res)
    }
    if (store._devtoolHook) {
      return res.catch(err => {
        store._devtoolHook.emit('vuex:error', err)
        throw err
      })
    } else {
      return res
    }
  })
}
```

- `store._actions[type]`存储结构和`_mutations`的一样，按命名空间来存储action
- 同样绑定this为store，但是第一个参数传入了一个对象`{ dispatch, commit, getters, state, rootGetters, rootState }`
- `{ dispatch, commit, getters, state }`是局部local的
- 运行后的结果是返回一个Promise

**registerGetter:**

```js
function registerGetter (store, type, rawGetter, local) {
  if (store._wrappedGetters[type]) {
    if (process.env.NODE_ENV !== 'production') {
      console.error(`[vuex] duplicate getter key: ${type}`)
    }
    return
  }
  store._wrappedGetters[type] = function wrappedGetter (store) {
    return rawGetter(
      local.state, // local state
      local.getters, // local getters
      store.state, // root state
      store.getters // root getters
    )
  }
}
```

- `store._wrappedGetters[type]`由于所有getters放在一个对象，结构和actions、mutations的结构就不一样了，就是一个对象
- 键值还是根据命名空间来生成
- 传入四个参数(local.state, local.getters, store.state, store.getters)；有命名空间，前面2个参数就是命名空间的，没有命名空间前面2个参数就是全局的

**resetStore:**

```js
function resetStore (store, hot) {
  store._actions = Object.create(null)
  store._mutations = Object.create(null)
  store._wrappedGetters = Object.create(null)
  store._modulesNamespaceMap = Object.create(null)
  const state = store.state
  // init all modules
  installModule(store, state, [], store._modules.root, true)
  // reset vm
  resetStoreVM(store, state, hot)
}
```

- 这个会重新创建Store，整体的，整体替换旧的Store
- 重新`installModule`和`resetStoreVM`

## class Store 的方法

**state():**

```js
  get state () {
    return this._vm._data.$$state
  }

  set state (v) {
    if (process.env.NODE_ENV !== 'production') {
      assert(false, `use store.replaceState() to explicit replace store state.`)
    }
  }
```

- 获取的state是直接从`store._vm._data.$$state`中获取
- 不可以直接设置state

**replaceState:**

```js
  replaceState (state) {
    this._withCommit(() => {
      this._vm._data.$$state = state
    })
  }
```

- 替换状态是直接替换`store._vm._data.$$state`

**commit:**

```js
  commit (_type, _payload, _options) {
    // check object-style commit
    const {
      type,
      payload,
      options
    } = unifyObjectStyle(_type, _payload, _options)

    const mutation = { type, payload }
    const entry = this._mutations[type]
    if (!entry) {
      if (process.env.NODE_ENV !== 'production') {
        console.error(`[vuex] unknown mutation type: ${type}`)
      }
      return
    }
    this._withCommit(() => {
      entry.forEach(function commitIterator (handler) {
        handler(payload)
      })
    })
    this._subscribers.forEach(sub => sub(mutation, this.state))

    if (
      process.env.NODE_ENV !== 'production' &&
      options && options.silent
    ) {
      console.warn(
        `[vuex] mutation type: ${type}. Silent option has been removed. ` +
        'Use the filter functionality in the vue-devtools'
      )
    }
  }
```

- 全局的`commit`，由于`_mutations`存储的是所有的`type`（包括模块的），这个commit可以`commit('shop/card')`只要命名路径对
- 调用完后，会发布`_subscribers`，遍历该数组调用回调

**dispatch:**

```js
  dispatch (_type, _payload) {
    // check object-style dispatch
    const {
      type,
      payload
    } = unifyObjectStyle(_type, _payload)

    const action = { type, payload }
    const entry = this._actions[type]
    if (!entry) {
      if (process.env.NODE_ENV !== 'production') {
        console.error(`[vuex] unknown action type: ${type}`)
      }
      return
    }

    this._actionSubscribers.forEach(sub => sub(action, this.state))

    return entry.length > 1
      ? Promise.all(entry.map(handler => handler(payload)))
      : entry[0](payload)
  }
```

- 全局`dispatch`，和`commit`一样，`_actions`存放的是所有的action的`type`（包括模块的）
- 调用完后会发布`_actionSubscribers`中订阅的回调函数
- `_actions[type]`是一个数组，可以一个`type`不同处理，也即`_actions[type] = [fn1, fn2, fn3, ...]`
- 如果出现一个type多个处理，就用`Promise.all`等到所有函数都调用完才统一处理（支持异步）

**subscribe:**

```js
  subscribe (fn) {
    return genericSubscribe(fn, this._subscribers)
  }
```

- 订阅`commit`，只要调用全局的`commit`就调用，模块内的`context.commit`也是会调用全部的`commit`

**subscribeAction:**

```js
  subscribeAction (fn) {
    return genericSubscribe(fn, this._actionSubscribers)
  }
```

- 订阅`action`，只要调用全局的`action`就调用，模块内的`context.dispatch`也是会调用全部的`dispatch`

**watch:**

```js
  watch (getter, cb, options) {
    if (process.env.NODE_ENV !== 'production') {
      assert(typeof getter === 'function', `store.watch only accepts a function.`)
    }
    return this._watcherVM.$watch(() => getter(this.state, this.getters), cb, options)
  }
```

- 借用`Vue`的`$watch`，观察想要监控的状态
- 用这个函数就相当于创建一个getter，不同时可动态创建

## 动态模块

- `registerModule`
- `unregisterModule`
- `hotUpdate`

**registerModule:**

```js
  registerModule (path, rawModule, options = {}) {
    if (typeof path === 'string') path = [path]

    if (process.env.NODE_ENV !== 'production') {
      assert(Array.isArray(path), `module path must be a string or an Array.`)
      assert(path.length > 0, 'cannot register the root module by using registerModule.')
    }

    this._modules.register(path, rawModule)
    installModule(this, this.state, path, this._modules.get(path), options.preserveState)
    // reset store to update getters...
    resetStoreVM(this, this.state)
  }
```

- 动态注册模块
- 注册一个模块会重新创建`Store`和`store._vm`
- `installModule` 和 `resetStoreVM`这两个函数很总要，涉及到性能，重点、重点

**unregisterModule:**

```js
  unregisterModule (path) {
    if (typeof path === 'string') path = [path]

    if (process.env.NODE_ENV !== 'production') {
      assert(Array.isArray(path), `module path must be a string or an Array.`)
    }

    this._modules.unregister(path)
    this._withCommit(() => {
      const parentState = getNestedState(this.state, path.slice(0, -1))
      Vue.delete(parentState, path[path.length - 1])
    })
    resetStore(this)
  }
```

- 注销模块
- `Vue.delete(parentState, path[path.length - 1])`和`resetStore(this)`
- 还是会重新创建`Store`和`_vm`

**hotUpdate:**

```js
  hotUpdate (newOptions) {
    this._modules.update(newOptions)
    resetStore(this, true)
  }
```

- 更新模块
- `resetStore(this, true)`，还是会重新创建`Store`和`_vm`

**很重要的3个函数:**

- `resetStore`：包含`resetStoreVM`和`installModule`
- `resetStoreVM`
- `installModule`