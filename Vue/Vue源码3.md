##Vue源码分析(3)

`src/core/util`文件夹还有3个很重要的文件

- `next-tick.js`
- `props.js`
- `options.js`

### next-tick.js

```js
const callbacks = []
let pending = false

function flushCallbacks () {
  pending = false
  const copies = callbacks.slice(0)
  callbacks.length = 0
  for (let i = 0; i < copies.length; i++) {
    copies[i]()
  }
}

// Here we have async deferring wrappers using both microtasks and (macro) tasks.
// In < 2.4 we used microtasks everywhere, but there are some scenarios where
// microtasks have too high a priority and fire in between supposedly
// sequential events (e.g. #4521, #6690) or even between bubbling of the same
// event (#6566). However, using (macro) tasks everywhere also has subtle problems
// when state is changed right before repaint (e.g. #6813, out-in transitions).
// Here we use microtask by default, but expose a way to force (macro) task when
// needed (e.g. in event handlers attached by v-on).
let microTimerFunc
let macroTimerFunc
let useMacroTask = false
```

- `callbacks`：下个event loop 要执行的所有程序，会遍历数组中所有的函数以及执行
- `pending`：记录状态，是个状态变量，确保同一时间只执行一个，避免原子性
- `flushCallbacks`:是遍历`callbacks`以及执行里面所有的函数，然后重置`callbacks`
- `callbacks.length = 0`这是一个重置数组非常好的技巧，而不是新创建一个新的数组来替换，旧的被垃圾机制回收
- `callbacks.slice(0)`克隆一份
- 定义可以加入microtasks或macrotasks队列的函数`macroTimerFunc`和`microTimerFunc`
- `useMacroTask`是决定`nextTick`是使用macroTimerFunc还是microTimerFunc，也即加入micro还是macro队列，默认是使用了`microTimerFunc`
- 如果加入了macro队列，那会在下一个event loop才会执行，加入了micro队列会在当前event loop执行
- `macroTimerFunc`和`microTimerFunc`共享`callbacks` ，这二者都有执行清空callbacks以及遍历执行里面的函数

```js
// Determine (macro) task defer implementation.
// Technically setImmediate should be the ideal choice, but it's only available
// in IE. The only polyfill that consistently queues the callback after all DOM
// events triggered in the same loop is by using MessageChannel.
/* istanbul ignore if */
if (typeof setImmediate !== 'undefined' && isNative(setImmediate)) {
  macroTimerFunc = () => {
    setImmediate(flushCallbacks)
  }
} else if (typeof MessageChannel !== 'undefined' && (
  isNative(MessageChannel) ||
  // PhantomJS
  MessageChannel.toString() === '[object MessageChannelConstructor]'
)) {
  const channel = new MessageChannel()
  const port = channel.port2
  channel.port1.onmessage = flushCallbacks
  macroTimerFunc = () => {
    port.postMessage(1)
  }
} else {
  /* istanbul ignore next */
  macroTimerFunc = () => {
    setTimeout(flushCallbacks, 0)
  }
}
```

- 定义`macroTimerFunc`，可以充当macro函数的比较多，最好的是`setImmediate`，然后`MessageChannel`的方式，最差的使用原生的定时器
- 其中`MessageChannel`和`setImmediate`要考虑兼容性，`MessageChannel`在移动端兼容性还是可以的
- `MessageChannel`还可以用在`Web Worker`
- `macroTimerFunc`加入的是macrotasks队列

```js
// Determine microtask defer implementation.
/* istanbul ignore next, $flow-disable-line */
if (typeof Promise !== 'undefined' && isNative(Promise)) {
  const p = Promise.resolve()
  microTimerFunc = () => {
    p.then(flushCallbacks)
    // in problematic UIWebViews, Promise.then doesn't completely break, but
    // it can get stuck in a weird state where callbacks are pushed into the
    // microtask queue but the queue isn't being flushed, until the browser
    // needs to do some other work, e.g. handle a timer. Therefore we can
    // "force" the microtask queue to be flushed by adding an empty timer.
    if (isIOS) setTimeout(noop)
  }
} else {
  // fallback to macro
  microTimerFunc = macroTimerFunc
}
```

- `microTimerFunc`主要使用Promise。如果浏览器不兼容就采用`macroTimerFunc`
- 由于microtasks队列是在当前evnet loop执行的，所以`nextTick`默认是使用`microTimerFunc`

**withMacroTask**

```js
/**
 * Wrap a function so that if any code inside triggers state change,
 * the changes are queued using a (macro) task instead of a microtask.
 */
export function withMacroTask (fn: Function): Function {
  return fn._withTask || (fn._withTask = function () {
    useMacroTask = true
    const res = fn.apply(null, arguments)
    useMacroTask = false
    return res
  })
}
```

- 由于`nextTick`默认是使用的`microTimerFunc`，如果想使用好一点的`macroTimerFunc`
- 用单例的方式给一个函数添加`_withTash`函数

**nextTick**

```js
export function nextTick (cb?: Function, ctx?: Object) {
  let _resolve
  callbacks.push(() => {
    if (cb) {
      try {
        cb.call(ctx)
      } catch (e) {
        handleError(e, ctx, 'nextTick')
      }
    } else if (_resolve) {
      _resolve(ctx)
    }
  })
  if (!pending) {
    pending = true
    if (useMacroTask) {
      macroTimerFunc()
    } else {
      microTimerFunc()
    }
  }
  // $flow-disable-line
  if (!cb && typeof Promise !== 'undefined') {
    return new Promise(resolve => {
      _resolve = resolve
    })
  }
}
```

```js
  if (typeof Promise !== 'undefined') {
    it('returns a Promise when provided no callback', done => {
      nextTick().then(done)
    })

    it('returns a Promise with a context argument when provided a falsy callback and an object', done => {
      const obj = {}
      nextTick(undefined, obj).then(ctx => {
        expect(ctx).toBe(obj)
        done()
      })
    })

    it('returned Promise should resolve correctly vs callback', done => {
      const spy = jasmine.createSpy()
      nextTick(spy)
      nextTick().then(() => {
        expect(spy).toHaveBeenCalled()
        done()
      })
    })
  }
```

- 如果没有添加`cb`参数或为空值，且环境中存在Promise，最终结果会是`Promise.resolve(ctx)`
- `pending`是保证`macroTimerFunc`或`microTimerFunc`是同步的，但是`callbacks`添加元素是可以异步的

### options.js

options主要采用了合并策略的方式，这个文件很多函数定义给是合并策略的     
options是一个很重要的文件，因为平时我们写的组件其实都是在写option，这个解读这个文件可以了解更多的细节

**resolveAsset**

```js
/**
 * Resolve an asset.
 * This function is used because child instances need access
 * to assets defined in its ancestor chain.
 */
export function resolveAsset (
  options: Object,
  type: string,
  id: string,
  warnMissing?: boolean
): any {
  /* istanbul ignore if */
  if (typeof id !== 'string') {
    return
  }
  const assets = options[type]
  // check local registration variations first
  if (hasOwn(assets, id)) return assets[id]
  const camelizedId = camelize(id)
  if (hasOwn(assets, camelizedId)) return assets[camelizedId]
  const PascalCaseId = capitalize(camelizedId)
  if (hasOwn(assets, PascalCaseId)) return assets[PascalCaseId]
  // fallback to prototype chain
  const res = assets[id] || assets[camelizedId] || assets[PascalCaseId]
  if (process.env.NODE_ENV !== 'production' && warnMissing && !res) {
    warn(
      'Failed to resolve ' + type.slice(0, -1) + ': ' + id,
      options
    )
  }
  return res
}
```

- `const assets = options[type]`，assets就是options属性的值
- 这里会用优先使用原先的id去查看`assets`是否有该属性，有则返回`assets[id]`
- id不匹配会以驼峰或者单词首字母大写的命名方式去匹配，都不匹配则报警告

**defaultStrat**

```js
/**
 * Default strategy.
 */
const defaultStrat = function (parentVal: any, childVal: any): any {
  return childVal === undefined
    ? parentVal
    : childVal
}
```

- 默认的策略是直接替换，不是合并，替换是我之前options有的属性，新的options没有，会被删掉，而合并会保留
- 合并是旧的options的字段名如果和新的options字段名一样，会被新的替换调，不一样的会保留

**validateComponentName、checkComponents**

```js
/**
 * Validate component names
 */
function checkComponents (options: Object) {
  for (const key in options.components) {
    validateComponentName(key)
  }
}

export function validateComponentName (name: string) {
  if (!/^[a-zA-Z][\w-]*$/.test(name)) {
    warn(
      'Invalid component name: "' + name + '". Component names ' +
      'can only contain alphanumeric characters and the hyphen, ' +
      'and must start with a letter.'
    )
  }
  if (isBuiltInTag(name) || config.isReservedTag(name)) {
    warn(
      'Do not use built-in or reserved HTML elements as component ' +
      'id: ' + name
    )
  }
}
```

- 主要是检测定义的组件名是否合法，组件名只能由字母和'-'组成，且不能是HTML的标签名如:`<div>`以及不能是Vue预先定义的组件名：如`<Component>`

**合并策略**

```js
/**
 * Option overwriting strategies are functions that handle
 * how to merge a parent option value and a child option
 * value into the final value.
 */
const strats = config.optionMergeStrategies
```

- 合并采用的是策略模式，`strats`的每个字段和options的主要字段一样，值是各个字段的预定合并函数

strats拥有的字段名

- el
- propsData
- data
- watch
- props
- methods
- inject
- computed
- provide

- components
- filters
- directives

- beforeCreate
- created
- beforeMount
- mounted
- beforeUpdate
- updated
- beforeDestroy
- destroyed
- activated
- deactivated
- errorCaptured

上面这个都对应一个合并函数，策略保存着所有的合并函数，真正合并的时候只需根据字段名来采用不同的合并方式

**el、propsData**

```js
/**
 * Options with restrictions
 */
if (process.env.NODE_ENV !== 'production') {
  strats.el = strats.propsData = function (parent, child, vm, key) {
    if (!vm) {
      warn(
        `option "${key}" can only be used during instance ` +
        'creation with the `new` keyword.'
      )
    }
    return defaultStrat(parent, child)
  }
}
```

- el和propsData采用的是默认的替换方式，el就是字符串，简单的基本数据类型替换即可

**mergeData**

```js
/**
 * Helper that recursively merges two data objects together.
 */
function mergeData (to: Object, from: ?Object): Object {
  if (!from) return to
  let key, toVal, fromVal
  const keys = Object.keys(from)
  for (let i = 0; i < keys.length; i++) {
    key = keys[i]
    toVal = to[key]
    fromVal = from[key]
    if (!hasOwn(to, key)) {
      set(to, key, fromVal)
    } else if (isPlainObject(toVal) && isPlainObject(fromVal)) {
      mergeData(toVal, fromVal)
    }
  }
  return to
}
```

- 主要合并两个对象，用于`data`中
- 由于这些数据是定于`data`中的，要添加`set(to, key, fromVal)`，这些合并是深度合并的

**mergeDataFn**

```js
/**
 * Data
 */
export function mergeDataOrFn (
  parentVal: any,
  childVal: any,
  vm?: Component
): ?Function {
  if (!vm) {
    // in a Vue.extend merge, both should be functions
    if (!childVal) {
      return parentVal
    }
    if (!parentVal) {
      return childVal
    }
    // when parentVal & childVal are both present,
    // we need to return a function that returns the
    // merged result of both functions... no need to
    // check if parentVal is a function here because
    // it has to be a function to pass previous merges.
    return function mergedDataFn () {
      return mergeData(
        typeof childVal === 'function' ? childVal.call(this, this) : childVal,
        typeof parentVal === 'function' ? parentVal.call(this, this) : parentVal
      )
    }
  } else {
    return function mergedInstanceDataFn () {
      // instance merge
      const instanceData = typeof childVal === 'function'
        ? childVal.call(vm, vm)
        : childVal
      const defaultData = typeof parentVal === 'function'
        ? parentVal.call(vm, vm)
        : parentVal
      if (instanceData) {
        return mergeData(instanceData, defaultData)
      } else {
        return defaultData
      }
    }
  }
}
```

- 考虑到继承的情况这个函数分成2种方式，如果传入`vm`就是继承，使用`mergedInstanceDataFn`否则使用`mergedDataFn`
- 定于`data`可以是对象也可以是函数，所以在合并之前要先判断data是不是函数，是函数要用函数执行后的值来合并，这里还涉及到mixins，别漏了mixins

**data**

```js
strats.data = function (
  parentVal: any,
  childVal: any,
  vm?: Component
): ?Function {
  if (!vm) {
    if (childVal && typeof childVal !== 'function') {
      process.env.NODE_ENV !== 'production' && warn(
        'The "data" option should be a function ' +
        'that returns a per-instance value in component ' +
        'definitions.',
        vm
      )

      return parentVal
    }
    return mergeDataOrFn(parentVal, childVal)
  }

  return mergeDataOrFn(parentVal, childVal, vm)
}
```

- 这里定义data的时候，最好使用函数的方式定义，如果在开发环境下使用对象的方式定义，会报上面的警告

**生命周期钩子**

```js
/**
 * Hooks and props are merged as arrays.
 */
function mergeHook (
  parentVal: ?Array<Function>,
  childVal: ?Function | ?Array<Function>
): ?Array<Function> {
  return childVal
    ? parentVal
      ? parentVal.concat(childVal)
      : Array.isArray(childVal)
        ? childVal
        : [childVal]
    : parentVal
}

LIFECYCLE_HOOKS.forEach(hook => {
  strats[hook] = mergeHook
})
```

- 生命周期钩子的合并，是把所有的钩子函数收集与一个数组中，执行钩子的时候，所有钩子函数都会执行
- 如`{ beforeCreated: [fn1, fn2, fn2, ...] }`

**components、filters、directives**

```js
/**
 * Assets
 *
 * When a vm is present (instance creation), we need to do
 * a three-way merge between constructor options, instance
 * options and parent options.
 */
function mergeAssets (
  parentVal: ?Object,
  childVal: ?Object,
  vm?: Component,
  key: string
): Object {
  const res = Object.create(parentVal || null)
  if (childVal) {
    process.env.NODE_ENV !== 'production' && assertObjectType(key, childVal, vm)
    return extend(res, childVal)
  } else {
    return res
  }
}

ASSET_TYPES.forEach(function (type) {
  strats[type + 's'] = mergeAssets
})
```

- `components、filters、directives`才有的合并是`extend`
- `extend`只是把另一个对象的所有属性值赋值给新的对象。如果是引用类型，只是复制一份引用，并不是克隆
- 由于是`extend`的，如果出现重名的，会被最后的替代

**watch**

```js
/**
 * Watchers.
 *
 * Watchers hashes should not overwrite one
 * another, so we merge them as arrays.
 */
strats.watch = function (
  parentVal: ?Object,
  childVal: ?Object,
  vm?: Component,
  key: string
): ?Object {
  // work around Firefox's Object.prototype.watch...
  if (parentVal === nativeWatch) parentVal = undefined
  if (childVal === nativeWatch) childVal = undefined
  /* istanbul ignore if */
  if (!childVal) return Object.create(parentVal || null)
  if (process.env.NODE_ENV !== 'production') {
    assertObjectType(key, childVal, vm)
  }
  if (!parentVal) return childVal
  const ret = {}
  extend(ret, parentVal)
  for (const key in childVal) {
    let parent = ret[key]
    const child = childVal[key]
    if (parent && !Array.isArray(parent)) {
      parent = [parent]
    }
    ret[key] = parent
      ? parent.concat(child)
      : Array.isArray(child) ? child : [child]
  }
  return ret
}
```

- watch不是替换，有多个同名的watch会收集到一个数组中
- watch的key对应的值都是数组，不是数组会自动转换成数组
- 比如有多个mixin，都定义了一个同名的watch，最后的效果是都会执行

**props、methods、inject、computed、provide**

```js
/**
 * Other object hashes.
 */
strats.props =
strats.methods =
strats.inject =
strats.computed = function (
  parentVal: ?Object,
  childVal: ?Object,
  vm?: Component,
  key: string
): ?Object {
  if (childVal && process.env.NODE_ENV !== 'production') {
    assertObjectType(key, childVal, vm)
  }
  if (!parentVal) return childVal
  const ret = Object.create(null)
  extend(ret, parentVal)
  if (childVal) extend(ret, childVal)
  return ret
}
strats.provide = mergeDataOrFn
```

- 除了`provide`其他的都是使用`extend`，`provide`的处理和`data`的一样，`provide要和inject一起使用，效果类似React的context`
- 由于是`extend`，出现重名的会被替代

**normalizeProps**

```js
/**
 * Ensure all props option syntax are normalized into the
 * Object-based format.
 */
function normalizeProps (options: Object, vm: ?Component) {
  const props = options.props
  if (!props) return
  const res = {}
  let i, val, name
  if (Array.isArray(props)) {
    i = props.length
    while (i--) {
      val = props[i]
      if (typeof val === 'string') {
        name = camelize(val)
        res[name] = { type: null }
      } else if (process.env.NODE_ENV !== 'production') {
        warn('props must be strings when using array syntax.')
      }
    }
  } else if (isPlainObject(props)) {
    for (const key in props) {
      val = props[key]
      name = camelize(key)
      res[name] = isPlainObject(val)
        ? val
        : { type: val }
    }
  } else if (process.env.NODE_ENV !== 'production') {
    warn(
      `Invalid value for option "props": expected an Array or an Object, ` +
      `but got ${toRawType(props)}.`,
      vm
    )
  }
  options.props = res
}
```

- 在组件中定义`props`只能是数组或者对象，如果是数组，值只能是字符串`['name', 'age']`，`{ type: null }`代表任何类型或者不检测类型
- 如果是对象，对象有两种方式`{ name: { default: 'MyDuty', type: String }, age: Number }`，对象一定要带类型，类型一定要用原生的构造函数，判断依据是判断是不是原生的String或Number等
- `props.name`和`props.age`
- 最终在程序内部使用的props的key都会格式化成驼峰的格式firstName

**normalizeInject**

```js
/**
 * Normalize all injections into Object-based format
 */
function normalizeInject (options: Object, vm: ?Component) {
  const inject = options.inject
  if (!inject) return
  const normalized = options.inject = {}
  if (Array.isArray(inject)) {
    for (let i = 0; i < inject.length; i++) {
      normalized[inject[i]] = { from: inject[i] }
    }
  } else if (isPlainObject(inject)) {
    for (const key in inject) {
      const val = inject[key]
      normalized[key] = isPlainObject(val)
        ? extend({ from: key }, val)
        : { from: val }
    }
  } else if (process.env.NODE_ENV !== 'production') {
    warn(
      `Invalid value for option "inject": expected an Array or an Object, ` +
      `but got ${toRawType(inject)}.`,
      vm
    )
  }
}
```

- `inject`有两种方式定义，一个数组一个对象，数组和`props`的一样
- 如果是对象，可以重写命名要注入的key或者key是`provide`的key，如果要重写命名，要加`form: keyName`，`keyName`一定存在于`provide`
- 对象的方式还可以是用默认值`default`，`default`可以是字符串也可以是函数

**normalizeDirectives**

```js
/**
 * Normalize raw function directives into object format.
 */
function normalizeDirectives (options: Object) {
  const dirs = options.directives
  if (dirs) {
    for (const key in dirs) {
      const def = dirs[key]
      if (typeof def === 'function') {
        dirs[key] = { bind: def, update: def }
      }
    }
  }
}
```

- 自定义指令有两种方式，一个是对象或者直接函数
- 这里的意思是，如果定义的是函数，bind和update都使用这个函数

**mergeOptions**

```js
/**
 * Merge two option objects into a new one.
 * Core utility used in both instantiation and inheritance.
 */
export function mergeOptions (
  parent: Object,
  child: Object,
  vm?: Component
): Object {
  if (process.env.NODE_ENV !== 'production') {
    checkComponents(child)
  }

  if (typeof child === 'function') {
    child = child.options
  }

  normalizeProps(child, vm)
  normalizeInject(child, vm)
  normalizeDirectives(child)
  const extendsFrom = child.extends
  if (extendsFrom) {
    parent = mergeOptions(parent, extendsFrom, vm)
  }
  if (child.mixins) {
    for (let i = 0, l = child.mixins.length; i < l; i++) {
      parent = mergeOptions(parent, child.mixins[i], vm)
    }
  }
  const options = {}
  let key
  for (key in parent) {
    mergeField(key)
  }
  for (key in child) {
    if (!hasOwn(parent, key)) {
      mergeField(key)
    }
  }
  function mergeField (key) {
    const strat = strats[key] || defaultStrat
    options[key] = strat(parent[key], child[key], vm, key)
  }
  return options
}
```

- 这个函数才是最终合并option的函数
- `child.extends`的`extends`只能有一个，效果和mixin一样
- 接下来合并mixins
- 最后就是合并option的各个字段，采用之前存好各个字段的合并函数的策略对象，option字段名和策略对象的字段名是一一对应的，也就是各个字段名采用各自的合并函数`mergeField`

### props.js

这个文件就是验证用户定义的prop和传入来的prop是否一致，类似React的`propTypes`    
主要是`validateProp`这个函数的实现


