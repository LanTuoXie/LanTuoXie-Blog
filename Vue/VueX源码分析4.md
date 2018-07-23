## VueX源码分析(4)

- `/module`
- `store.js`

#### /module/module.js

```js
import { forEachValue } from '../util'

// Base data struct for store's module, package with some attribute and method
export default class Module {
  constructor (rawModule, runtime) {
    this.runtime = runtime
    // Store some children item
    this._children = Object.create(null)
    // Store the origin module object which passed by programmer
    this._rawModule = rawModule
    const rawState = rawModule.state

    // Store the origin module's state
    this.state = (typeof rawState === 'function' ? rawState() : rawState) || {}
  }

  get namespaced () {
    return !!this._rawModule.namespaced
  }

  addChild (key, module) {
    this._children[key] = module
  }

  removeChild (key) {
    delete this._children[key]
  }

  getChild (key) {
    return this._children[key]
  }

  update (rawModule) {
    this._rawModule.namespaced = rawModule.namespaced
    if (rawModule.actions) {
      this._rawModule.actions = rawModule.actions
    }
    if (rawModule.mutations) {
      this._rawModule.mutations = rawModule.mutations
    }
    if (rawModule.getters) {
      this._rawModule.getters = rawModule.getters
    }
  }

  forEachChild (fn) {
    forEachValue(this._children, fn)
  }

  forEachGetter (fn) {
    if (this._rawModule.getters) {
      forEachValue(this._rawModule.getters, fn)
    }
  }

  forEachAction (fn) {
    if (this._rawModule.actions) {
      forEachValue(this._rawModule.actions, fn)
    }
  }

  forEachMutation (fn) {
    if (this._rawModule.mutations) {
      forEachValue(this._rawModule.mutations, fn)
    }
  }
}

```

解析：
- `Module`是store模块的类，基本模块的静态属性和一些方法
- `rawModule`就是我们定义的模块对象`{ namespaced, state, actions, mutations, getters }`
- `this.state = (typeof rawState === 'function' ? rawState() : rawState) || {}`定义状态可以用函数的方式定义的，这里要先判断状态是不是函数，是函数要执行函数后的值。
- `_children`记录嵌套模块的模块`{ state, modules: { modules: {} } }`
- `namespaced`就是是否使用命名空间
- `addChild、removeChild、getChild`是嵌套模块的一些操作方法
- 还有一些遍历的方法但是只有`children、getters、actions、mutations`的遍历，没有`this.state`，那个`update`模块也是，不会更新`state`
- 更新只更新`namespaced、actions、mutations、getters`

#### /module/module-collection.js

主要类`ModuleCollection`，还有一些辅助函数，先分析辅助函数再分析主要类。
主要将所有模块合并的类。

**断言函数（只在开发者环境起作用）**

```js
const functionAssert = {
  assert: value => typeof value === 'function',
  expected: 'function'
}

const objectAssert = {
  assert: value => typeof value === 'function' ||
    (typeof value === 'object' && typeof value.handler === 'function'),
  expected: 'function or object with "handler" function'
}

const assertTypes = {
  getters: functionAssert,
  mutations: functionAssert,
  actions: objectAssert
}

function assertRawModule (path, rawModule) {
  Object.keys(assertTypes).forEach(key => {
    if (!rawModule[key]) return

    const assertOptions = assertTypes[key]

    forEachValue(rawModule[key], (value, type) => {
      assert(
        assertOptions.assert(value),
        makeAssertionMessage(path, key, type, value, assertOptions.expected)
      )
    })
  })
}

function makeAssertionMessage (path, key, type, value, expected) {
  let buf = `${key} should be ${expected} but "${key}.${type}"`
  if (path.length > 0) {
    buf += ` in module "${path.join('.')}"`
  }
  buf += ` is ${JSON.stringify(value)}.`
  return buf
}
```

解析：
- `path`是嵌套模块的名称。如根模块为`[]`，嵌套模块`shop/card`为`['shop', 'card']`。主要功能是模块的寻址路径，可以根据这个路径获取该模块。
- `makeAssertionMessage(path, key, type, value, expected)`中的`key`就是我们自定义模块的字段：如`state、mutations`等，这个断言就是判断我们定义的字段命是否符合要求。

**ModuleCollection**

```js
export default class ModuleCollection {
  constructor (rawRootModule) {
    // register root module (Vuex.Store options)
    this.register([], rawRootModule, false)
  }

  get (path) {
    return path.reduce((module, key) => {
      return module.getChild(key)
    }, this.root)
  }

  getNamespace (path) {
    let module = this.root
    return path.reduce((namespace, key) => {
      module = module.getChild(key)
      return namespace + (module.namespaced ? key + '/' : '')
    }, '')
  }

  update (rawRootModule) {
    update([], this.root, rawRootModule)
  }

  register (path, rawModule, runtime = true) {
    if (process.env.NODE_ENV !== 'production') {
      assertRawModule(path, rawModule)
    }

    const newModule = new Module(rawModule, runtime)
    if (path.length === 0) {
      this.root = newModule
    } else {
      const parent = this.get(path.slice(0, -1))
      parent.addChild(path[path.length - 1], newModule)
    }

    // register nested modules
    if (rawModule.modules) {
      forEachValue(rawModule.modules, (rawChildModule, key) => {
        this.register(path.concat(key), rawChildModule, runtime)
      })
    }
  }

  unregister (path) {
    const parent = this.get(path.slice(0, -1))
    const key = path[path.length - 1]
    if (!parent.getChild(key).runtime) return

    parent.removeChild(key)
  }
}

function update (path, targetModule, newModule) {
  if (process.env.NODE_ENV !== 'production') {
    assertRawModule(path, newModule)
  }

  // update target module
  targetModule.update(newModule)

  // update nested modules
  if (newModule.modules) {
    for (const key in newModule.modules) {
      if (!targetModule.getChild(key)) {
        if (process.env.NODE_ENV !== 'production') {
          console.warn(
            `[vuex] trying to add a new module '${key}' on hot reloading, ` +
            'manual reload is needed'
          )
        }
        return
      }
      update(
        path.concat(key),
        targetModule.getChild(key),
        newModule.modules[key]
      )
    }
  }
}
```

解析：
- 主要功能是将所有模块合并起来，以及注册和注销所有模块。
- `register`就是将我们自己定义的对象模块`new Module(自己定义的对象模块)`，这个注册可以递归地注册所有模块，包括嵌套的
- `unregister`是`module.js`文件中的`delete this._children[key]`是直接调用delete删除
- `this.root`就是最外层的modules