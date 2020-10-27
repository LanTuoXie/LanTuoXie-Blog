# Vue源码observer

`./observer`文件夹一共有如下文件

- `array.js`
- `dep.js`
- `index.js`
- `scheduler.js`
- `traverse.js`
- `watcher.js`

首先说说数据拦截的方式

```js
/**
 * Define a property.
 */
export function def (obj: Object, key: string, val: any, enumerable?: boolean) {
  Object.defineProperty(obj, key, {
    value: val,
    enumerable: !!enumerable,
    writable: true,
    configurable: true
  })
}
```

- 能够进行数据拦截的数据类型是`Object`类型，也是为什么data要返回一个对象
- 最后那个`enumerable`是否可以遍历属性，默认值是false，也就是如果不设为true，那么设置的属性是隐藏属性，`in`操作符无法获取`Object.keys`也无法获取
- 但是`Object.getOwnPropertyNames`可以获取到这些不可遍历的属性

接下来的内容由简到难，先从关联性比较低的文件开始分析。

## `array.js`

拦截一些原生Array的方法，进行ob，如果改变了，更新依赖

- `push`
- `pop`
- `shift`
- `unshift`
- `splice`
- `sort`
- `reverse`

一个拥有`__ob__`的数组，如果调用上面的方法，有自动响应的效果。如果是添加元素，元素还会自动成为响应式元素

```js
const arrayProto = Array.prototype
export const arrayMethods = Object.create(arrayProto)

const methodsToPatch = [
  'push',
  'pop',
  'shift',
  'unshift',
  'splice',
  'sort',
  'reverse'
]

/**
 * Intercept mutating methods and emit events
 */
methodsToPatch.forEach(function (method) {
  // cache original method
  const original = arrayProto[method]
  def(arrayMethods, method, function mutator (...args) {
    const result = original.apply(this, args)
    const ob = this.__ob__
    let inserted
    switch (method) {
      case 'push':
      case 'unshift':
        inserted = args
        break
      case 'splice':
        inserted = args.slice(2)
        break
    }
    if (inserted) ob.observeArray(inserted)
    // notify change
    ob.dep.notify()
    return result
  })
})
```

- 可以看出，上面那7个数组的函数，是在不改变原生方法的基础上克隆一个可以响应数据变化的7个新数组函数
- 其实就是使用装饰者模式，给原生数据方法扩展一个检测数据变化和依赖的效果

```js
  def(arrayMethods, method, function mutator (...args) {
    const result = original.apply(this, args)
    return result
  })
```

- 上面的效果就是原生数组的那些方法的效果，或者可以认为就是原生的数组方法

```js
    const ob = this.__ob__
    let inserted
    switch (method) {
      case 'push':
      case 'unshift':
        inserted = args
        break
      case 'splice':
        inserted = args.slice(2)
        break
    }
    if (inserted) ob.observeArray(inserted)
    // notify change
    ob.dep.notify()
```

- 这里的`this`就是经过了`observer`函数处理的数组，这时候就应该拥有属性`__ob__`
- `push、unshitf、splice`都是向数组添加元素，使用这3个方法添加的元素，那些元素都会转化成`ob属性`（响应式属性）
- `ob.dep.notify()`主要是上面7个方法调用，通知所有订阅这个数组的依赖，这个数组已经改变了

### `traverse.js`

```js
const seenObjects = new Set()

/**
 * Recursively traverse an object to evoke all converted
 * getters, so that every nested property inside the object
 * is collected as a "deep" dependency.
 */
export function traverse (val: any) {
  _traverse(val, seenObjects)
  seenObjects.clear()
}

function _traverse (val: any, seen: SimpleSet) {
  let i, keys
  const isA = Array.isArray(val)
  if ((!isA && !isObject(val)) || Object.isFrozen(val) || val instanceof VNode) {
    return
  }
  if (val.__ob__) {
    const depId = val.__ob__.dep.id
    if (seen.has(depId)) {
      return
    }
    seen.add(depId)
  }
  if (isA) {
    i = val.length
    while (i--) _traverse(val[i], seen)
  } else {
    keys = Object.keys(val)
    i = keys.length
    while (i--) _traverse(val[keys[i]], seen)
  }
}
```

- 这个函数的主要目的是触发`getter`，添加`seenObjects`的目的是避免引用嵌套引用的情况
- 递归的出口：`不是数组、不是对象、是冻结的对象、是VNode`以及`seen.has(depId)`，出现相同`depId`也就是引用嵌套引用（自己引用自己或者多个相同引用）
- 递归的只有是数组或者对象的数据类型，然后递归访问它们的属性，也即触发`getter`
- 最主要的还是递归地触发所有`getter`，且支持嵌套引用的情况`seen.has(depId)`，这个递归出口的意思是，这些`getter`已经触发过了，还有避免嵌套引用的死循环（自己引用自己）
- 在递归引用类型的时候，要考虑的一个出口就是引用类型引用自身

## `dep.js`

```js
let uid = 0

/**
 * A dep is an observable that can have multiple
 * directives subscribing to it.
 */
export default class Dep {
  static target: ?Watcher;
  id: number;
  subs: Array<Watcher>;

  constructor () {
    this.id = uid++
    this.subs = []
  }

  addSub (sub: Watcher) {
    this.subs.push(sub)
  }

  removeSub (sub: Watcher) {
    remove(this.subs, sub)
  }

  depend () {
    if (Dep.target) {
      Dep.target.addDep(this)
    }
  }

  notify () {
    // stabilize the subscriber list first
    const subs = this.subs.slice()
    for (let i = 0, l = subs.length; i < l; i++) {
      subs[i].update()
    }
  }
}
```

- `uid`就是之前的`depId`
- 这是一个存储结构，存储订阅者或者观察者`Watcher`
- 只要是`Observable Data`可观察数据，也就是经过`observer`函数处理的对象，都有一个`dep`，用于存储谁订阅了我，或者谁引用了我（组件）
- `subs`存储的是`Watcher`，存储结构是数组
- `notify`就是发布，告诉所有订阅者我已经改变或者更新了，所有`Watcher`都有一个`update`函数
- 如果`Watcher`要实时同步订阅的数据，要实现一个`update`函数，当`dep`更新时，会自动调用`update`

```js
// the current target watcher being evaluated.
// this is globally unique because there could be only one
// watcher being evaluated at any time.
Dep.target = null
const targetStack = []

export function pushTarget (_target: ?Watcher) {
  if (Dep.target) targetStack.push(Dep.target)
  Dep.target = _target
}

export function popTarget () {
  Dep.target = targetStack.pop()
}
```

- `Dep.target`就是`Watcher`，记录的是当前和`Dep`互动的`Watcher`，任何时候都只能有唯一一个`Watcher`和`Dep`互动
- 添加的栈结构就是要确保在任何时候，和`Dep`互动，只能有一个`Watcher`，不能有多个
- 为什么使用栈结构，不使用队列？到要用到这里的时候再说，留个疑念

## `watcher.js`

根据文件名就知道，这是一个观察者，拥有订阅的功能    
观察者的功能：

- 解析一个表达式`expression`
- 收集依赖`collects dependencies`
- 当`expression`改变的时候执行回调`callback`
- 主要用于`$watch()`和指令上如`v-bind v-on 等`

```js
let uid = 0

/**
 * A watcher parses an expression, collects dependencies,
 * and fires callback when the expression value changes.
 * This is used for both the $watch() api and directives.
 */
export default class Watcher {
  vm: Component;
  expression: string;
  cb: Function;
  id: number;
  deep: boolean;
  user: boolean;
  computed: boolean;
  sync: boolean;
  dirty: boolean;
  active: boolean;
  dep: Dep;
  deps: Array<Dep>;
  newDeps: Array<Dep>;
  depIds: SimpleSet;
  newDepIds: SimpleSet;
  before: ?Function;
  getter: Function;
  value: any;

  constructor (
    vm: Component,
    expOrFn: string | Function,
    cb: Function,
    options?: ?Object,
    isRenderWatcher?: boolean
  ) {
    this.vm = vm
    if (isRenderWatcher) {
      vm._watcher = this
    }
    vm._watchers.push(this)
    // options
    if (options) {
      this.deep = !!options.deep
      this.user = !!options.user
      this.computed = !!options.computed
      this.sync = !!options.sync
      this.before = options.before
    } else {
      this.deep = this.user = this.computed = this.sync = false
    }
    this.cb = cb
    this.id = ++uid // uid for batching
    this.active = true
    this.dirty = this.computed // for computed watchers
    this.deps = []
    this.newDeps = []
    this.depIds = new Set()
    this.newDepIds = new Set()
    this.expression = process.env.NODE_ENV !== 'production'
      ? expOrFn.toString()
      : ''
    // parse expression for getter
    if (typeof expOrFn === 'function') {
      this.getter = expOrFn
    } else {
      this.getter = parsePath(expOrFn)
      if (!this.getter) {
        this.getter = function () {}
        process.env.NODE_ENV !== 'production' && warn(
          `Failed watching path: "${expOrFn}" ` +
          'Watcher only accepts simple dot-delimited paths. ' +
          'For full control, use a function instead.',
          vm
        )
      }
    }
    if (this.computed) {
      this.value = undefined
      this.dep = new Dep()
    } else {
      this.value = this.get()
    }
  }

  /**
   * Evaluate the getter, and re-collect dependencies.
   */
  get () {
    pushTarget(this)
    let value
    const vm = this.vm
    try {
      value = this.getter.call(vm, vm)
    } catch (e) {
      if (this.user) {
        handleError(e, vm, `getter for watcher "${this.expression}"`)
      } else {
        throw e
      }
    } finally {
      // "touch" every property so they are all tracked as
      // dependencies for deep watching
      if (this.deep) {
        traverse(value)
      }
      popTarget()
      this.cleanupDeps()
    }
    return value
  }

  /**
   * Add a dependency to this directive.
   */
  addDep (dep: Dep) {
    const id = dep.id
    if (!this.newDepIds.has(id)) {
      this.newDepIds.add(id)
      this.newDeps.push(dep)
      if (!this.depIds.has(id)) {
        dep.addSub(this)
      }
    }
  }

  /**
   * Clean up for dependency collection.
   */
  cleanupDeps () {
    let i = this.deps.length
    while (i--) {
      const dep = this.deps[i]
      if (!this.newDepIds.has(dep.id)) {
        dep.removeSub(this)
      }
    }
    let tmp = this.depIds
    this.depIds = this.newDepIds
    this.newDepIds = tmp
    this.newDepIds.clear()
    tmp = this.deps
    this.deps = this.newDeps
    this.newDeps = tmp
    this.newDeps.length = 0
  }

  /**
   * Subscriber interface.
   * Will be called when a dependency changes.
   */
  update () {
    /* istanbul ignore else */
    if (this.computed) {
      // A computed property watcher has two modes: lazy and activated.
      // It initializes as lazy by default, and only becomes activated when
      // it is depended on by at least one subscriber, which is typically
      // another computed property or a component's render function.
      if (this.dep.subs.length === 0) {
        // In lazy mode, we don't want to perform computations until necessary,
        // so we simply mark the watcher as dirty. The actual computation is
        // performed just-in-time in this.evaluate() when the computed property
        // is accessed.
        this.dirty = true
      } else {
        // In activated mode, we want to proactively perform the computation
        // but only notify our subscribers when the value has indeed changed.
        this.getAndInvoke(() => {
          this.dep.notify()
        })
      }
    } else if (this.sync) {
      this.run()
    } else {
      queueWatcher(this)
    }
  }

  /**
   * Scheduler job interface.
   * Will be called by the scheduler.
   */
  run () {
    if (this.active) {
      this.getAndInvoke(this.cb)
    }
  }

  getAndInvoke (cb: Function) {
    const value = this.get()
    if (
      value !== this.value ||
      // Deep watchers and watchers on Object/Arrays should fire even
      // when the value is the same, because the value may
      // have mutated.
      isObject(value) ||
      this.deep
    ) {
      // set new value
      const oldValue = this.value
      this.value = value
      this.dirty = false
      if (this.user) {
        try {
          cb.call(this.vm, value, oldValue)
        } catch (e) {
          handleError(e, this.vm, `callback for watcher "${this.expression}"`)
        }
      } else {
        cb.call(this.vm, value, oldValue)
      }
    }
  }

  /**
   * Evaluate and return the value of the watcher.
   * This only gets called for computed property watchers.
   */
  evaluate () {
    if (this.dirty) {
      this.value = this.get()
      this.dirty = false
    }
    return this.value
  }

  /**
   * Depend on this watcher. Only for computed property watchers.
   */
  depend () {
    if (this.dep && Dep.target) {
      this.dep.depend()
    }
  }

  /**
   * Remove self from all dependencies' subscriber list.
   */
  teardown () {
    if (this.active) {
      // remove self from vm's watcher list
      // this is a somewhat expensive operation so we skip it
      // if the vm is being destroyed.
      if (!this.vm._isBeingDestroyed) {
        remove(this.vm._watchers, this)
      }
      let i = this.deps.length
      while (i--) {
        this.deps[i].removeSub(this)
      }
      this.active = false
    }
  }
}
```

Watcher的静态属性：

- `let uid`是观察者的`id`
- `vm`：Vue的组件
- `expression`：表达式，也就是v-bind 或者`:name='MyDuty'`这些指令的表达式
- `cb`：回调

Watcher的构造函数：

- `isRenderWatcher`：是渲染函数`render()`，也就是Vue组件有一个专门管渲染的Watcher`vm._watcher`
- `vm._watchers`：保存当前Vue组件所有的Watcher
