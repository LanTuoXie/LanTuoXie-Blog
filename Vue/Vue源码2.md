# Vue源码分析(2)

由于工具是整个`./src/core`文件夹很多文件使用到，且是相对独立，最应该优先分析。
首先看`./src/core/util`整个文件下的`index.js`
由于用到`./src/core/config.js`全局配置文件，`config`只要配置下面这3部分

- user
- platform
- legacy

可以把`config.js`当初`core`文件夹的全局参数，每个都有初始值，且`option?`，`no、noop、identity`用于一些配置的初始值

```js
/* @flow */

export * from 'shared/util'
export * from './lang'
export * from './env'
export * from './options'
export * from './debug'
export * from './props'
export * from './error'
export * from './next-tick'
export { defineReactive } from '../observer/index'

```

- 全局工具`shared/util`在上一篇
- `env、debug、error`：第一个是主要判断用户的环境，是浏览器还是node或者移动端等；debug每个应用都有；还有报错error
- `lang`：语言方面用到的工具函数
- `options`和`props`是很重要，设置到`Vue`创建
- `next-tick`主要是`Vue.nextTick()`函数
- 还有一个`perf.js`这个主要是包装`window.performance`，只存在于浏览器环境，要检测

## `env.js`

这个文件很值得研究，有很多判断环境的方法

**hasProto:**

```js
// can we use __proto__?
export const hasProto = '__proto__' in {}
```

- 能否使用`__proto__`

**一些浏览器环境:**

```js
// Browser environment sniffing
export const inBrowser = typeof window !== 'undefined'
export const inWeex = typeof WXEnvironment !== 'undefined' && !!WXEnvironment.platform
export const weexPlatform = inWeex && WXEnvironment.platform.toLowerCase()
export const UA = inBrowser && window.navigator.userAgent.toLowerCase()
export const isIE = UA && /msie|trident/.test(UA)
export const isIE9 = UA && UA.indexOf('msie 9.0') > 0
export const isEdge = UA && UA.indexOf('edge/') > 0
export const isAndroid = (UA && UA.indexOf('android') > 0) || (weexPlatform === 'android')
export const isIOS = (UA && /iphone|ipad|ipod|ios/.test(UA)) || (weexPlatform === 'ios')
export const isChrome = UA && /chrome\/\d+/.test(UA) && !isEdge
```

- 由于`node`是没有`window`的，可以由`window`来判断是浏览器还是Node环境
- `inWeex`和`weexPlatform`是用来判断是不是`weex`环境
- `UA`是判断当前浏览器是什么浏览器的最好依据，`userAgent`报错的浏览器的信息
- `isIE、isIE9、isEdge`根据浏览器内核来判断是那款IE，Edge的内核`edge`，之前的IE内核是`msie或者trident`
- `android`和`ios`判断
- `isChrome`有点特殊，由于谷歌之前的内核是`webkit`但是很多浏览器都带webkit，主要判断`chrome`

**nativeWatch:**

```js
// Firefox has a "watch" function on Object.prototype...
export const nativeWatch = ({}).watch
```

- 火狐特有的watch

**supportPassive:**

```js
export let supportsPassive = false
if (inBrowser) {
  try {
    const opts = {}
    Object.defineProperty(opts, 'passive', ({
      get () {
        /* istanbul ignore next */
        supportsPassive = true
      }
    }: Object)) // https://github.com/facebook/flow/issues/285
    window.addEventListener('test-passive', null, opts)
  } catch (e) {}
}
```

- 这里主要用`try catch`来实例，`passive`是定义事件的时候使用了`event.preventDefault()`
- 如果不是通过`window.addEventListener('test-passive', eventHandler, { passive: false })`方式定义事件处理程序，在一些高版本的浏览器会警告，
- 也就是说，如果要想使用`event.preventDefault()`要定义事件处理程序时要设置`passive`为`false`否则不会阻止浏览器的默认行为
- 由于这里定义的事件处理程序时`null`，可以不用`removeEventListener`

**devtools:**

```js
// detect devtools
export const devtools = inBrowser && window.__VUE_DEVTOOLS_GLOBAL_HOOK__
```

- 浏览器插件，调试Vue的浏览器插件，如谷歌的Vue调试插件

**isNative:**

```js
export function isNative (Ctor: any): boolean {
  return typeof Ctor === 'function' && /native code/.test(Ctor.toString())
}
```

- 非常有用的一个判断方法，用于判断构造函数是不是原生的构造函数
- 一些原生的构造函数如：`Object、Function、Array、Boolean`等，还有ES6的`Map、Set`
- 所以可以用来判断当前环境是否支持ES6等新的功能和特性

**hasSymbol:**

```js
export const hasSymbol =
  typeof Symbol !== 'undefined' && isNative(Symbol) &&
  typeof Reflect !== 'undefined' && isNative(Reflect.ownKeys)
```

- 判断当前环境是否支持`Symbol`，判断的同时还判断`Reflect`，`Reflect`主要管理对象`Object`的特性
- `Reflect.ownKeys`可以拿到所有的`keys`包括`Symbol`和一些不能遍历的属性，以及原型链`prototype`

**_Set:**

```js
let _Set
/* istanbul ignore if */ // $flow-disable-line
if (typeof Set !== 'undefined' && isNative(Set)) {
  // use native Set when available.
  _Set = Set
} else {
  // a non-standard Set polyfill that only works with primitive keys.
  _Set = class Set implements SimpleSet {
    set: Object;
    constructor () {
      this.set = Object.create(null)
    }
    has (key: string | number) {
      return this.set[key] === true
    }
    add (key: string | number) {
      this.set[key] = true
    }
    clear () {
      this.set = Object.create(null)
    }
  }
}

interface SimpleSet {
  has(key: string | number): boolean;
  add(key: string | number): mixed;
  clear(): void;
}
```

- 先判断支不支持`Set`这个ES6的数据结构，数据类型吧，不支持自己创建一个简单
- `Object.create(null)`创建一个空对象，不包括原型链，一般创建一个Set或者Map的方式都是使用这个创建一个空对象作为基本的存储结构

### `debug.js`

这个主要是用于调试的，报错的时候，找到出错的组件名，从下到上递归收集组件树的信息（直到root）

- `repeat`：创建重复的字符串
- `warn`：警告
- `tip`：提示
- `generateComponentTrace`：收集组件树，树的基本数据结构是数组
- `formatComponentName`：格式化组件名为`<componentName>`好查找错误的组件

**repeat:**

```js
  const repeat = (str, n) => {
    let res = ''
    while (n) {
      if (n % 2 === 1) res += str
      if (n > 1) str += str
      n >>= 1
    }
    return res
  }
```

- 这个算法很有意思，打算详细分析一下
- `n >>= 1`整体数值除以2，如果是`1 >>= 1`返回0
- `while (n) { n >>= 1 }` 出口就是`1 >>= 1`
- `| 16 | 8 | 4 | 2 | 1 |`5位二进制`(1, 1 ,1 ,1 ,1) = 31`
- 在2进制中，只有最后一位`| 1 |`才是奇数，像`| 2 |、| 4 |`等等这些位数都是偶数，如果一个数值的`| 1 |`是1则是奇数，是0则是偶数
- 在计算机中判断一个数值是奇数还是偶数的依据的判断`| 1 |`是1还是0。`num % 2`运算其实就是`num 的 | 1 |`的值
- 如`10的二进制'1010'`一共四位，上面循环体一共遍历4次
- `str += str`一直在迭代，重复的次数为`2^n`
- `n % 2 === 1`只有奇偶位为1的时候`res`才收集`str`，而只有`| 8 |和| 2 |`位才为1，所以`res`迭代的次数只有2次，而`str`这个时候的重复次有由`8 + 2`刚好重复10次

**warn、tip:**

```js
  warn = (msg, vm) => {
    const trace = vm ? generateComponentTrace(vm) : ''

    if (config.warnHandler) {
      config.warnHandler.call(null, msg, vm, trace)
    } else if (hasConsole && (!config.silent)) {
      console.error(`[Vue warn]: ${msg}${trace}`)
    }
  }
  
  tip = (msg, vm) => {
    if (hasConsole && (!config.silent)) {
      console.warn(`[Vue tip]: ${msg}` + (
        vm ? generateComponentTrace(vm) : ''
      ))
    }
  }
```

- 优先使用配置的`warnHandler`没有采用`console.error`
- 一些警告，一般是定义组件的某些属性不规范或定义错误，给出的一些警告，警告的同时带上出现警告问题的是那个组件
- 组件轨迹就是当前组件到根组件的那些组件名，方便开发者调试，根据组件名轨迹，快速定位有问题的组件
- 如果没有`vm`就是普通的警告
- `tip`：配置的`config.silent`，要开启才会提示

**formatComponentName:**

```js
  formatComponentName = (vm, includeFile) => {
    if (vm.$root === vm) {
      return '<Root>'
    }
    const options = typeof vm === 'function' && vm.cid != null
      ? vm.options
      : vm._isVue
        ? vm.$options || vm.constructor.options
        : vm || {}
    let name = options.name || options._componentTag
    const file = options.__file
    if (!name && file) {
      const match = file.match(/([^/\\]+)\.vue$/)
      name = match && match[1]
    }

    return (
      (name ? `<${classify(name)}>` : `<Anonymous>`) +
      (file && includeFile !== false ? ` at ${file}` : '')
    )
  }
```

单元测试用例

```js
  it('properly format component names', () => {
    const vm = new Vue()
    expect(formatComponentName(vm)).toBe('<Root>')

    vm.$root = null
    vm.$options.name = 'hello-there'
    expect(formatComponentName(vm)).toBe('<HelloThere>')

    vm.$options.name = null
    vm.$options._componentTag = 'foo-bar-1'
    expect(formatComponentName(vm)).toBe('<FooBar1>')

    vm.$options._componentTag = null
    vm.$options.__file = '/foo/bar/baz/SomeThing.vue'
    expect(formatComponentName(vm)).toBe(`<SomeThing> at ${vm.$options.__file}`)
    expect(formatComponentName(vm, false)).toBe('<SomeThing>')

    vm.$options.__file = 'C:\\foo\\bar\\baz\\windows_file.vue'
    expect(formatComponentName(vm)).toBe(`<WindowsFile> at ${vm.$options.__file}`)
    expect(formatComponentName(vm, false)).toBe('<WindowsFile>')
  })
```

- 就是格式化组件名成`<ComponentName>`第一个字符都格式化大写，如果没有name的就`<Anonymous>`
- 如果那个组件存有文件的信息，还会拼上文件信息

**generateComponentTrace:**

```js
  generateComponentTrace = vm => {
    if (vm._isVue && vm.$parent) {
      const tree = []
      let currentRecursiveSequence = 0
      while (vm) {
        if (tree.length > 0) {
          const last = tree[tree.length - 1]
          if (last.constructor === vm.constructor) {
            currentRecursiveSequence++
            vm = vm.$parent
            continue
          } else if (currentRecursiveSequence > 0) {
            tree[tree.length - 1] = [last, currentRecursiveSequence]
            currentRecursiveSequence = 0
          }
        }
        tree.push(vm)
        vm = vm.$parent
      }
      return '\n\nfound in\n\n' + tree
        .map((vm, i) => `${
          i === 0 ? '---> ' : repeat(' ', 5 + i * 2)
        }${
          Array.isArray(vm)
            ? `${formatComponentName(vm[0])}... (${vm[1]} recursive calls)`
            : formatComponentName(vm)
        }`)
        .join('\n')
    } else {
      return `\n\n(found in ${formatComponentName(vm)})`
    }
  }
```

测试用例

```js
  it('generate correct component hierarchy trace (recursive)', () => {
    let i = 0
    const one = {
      name: 'one',
      render: h => i++ < 5 ? h(one) : h(two)
    }
    const two = {
      name: 'two',
      render: h => h(three)
    }
    const three = {
      name: 'three'
    }
    new Vue({
      render: h => h(one)
    }).$mount()

    expect(
      `Failed to mount component: template or render function not defined.

found in

---> <Three>
       <Two>
         <One>... (5 recursive calls)
           <Root>`
    ).toHaveBeenWarned()
  })
```

- 记录出错组件的轨迹，递归地收集组件间的关系，以及格式化组件名
- 如果`vm`是数组，有递归渲染自己，如单元测试案例中的`render: h => i++ < 5 ? h(one) : h(two)`，递归渲染5次
- `last.constructor === vm.constructor`这个就是判断递归的依据，递归的都是由同一个构造函数创建的，所以用构造函数是否一样来判断是否处于递归中

### error.js

主要有三个函数

- `handleError`：主要的处理错误的函数
- `globalHandleError`：有没有在配置设置了`config.errorHandler`，有则用配置的，否则用`logError`
- `logError`：`主要是debug.js`中的`warn函数`

```js
export function handleError (err: Error, vm: any, info: string) {
  if (vm) {
    let cur = vm
    while ((cur = cur.$parent)) {
      const hooks = cur.$options.errorCaptured
      if (hooks) {
        for (let i = 0; i < hooks.length; i++) {
          try {
            const capture = hooks[i].call(cur, err, vm, info) === false
            if (capture) return
          } catch (e) {
            globalHandleError(e, cur, 'errorCaptured hook')
          }
        }
      }
    }
  }
  globalHandleError(err, vm, info)
}

function globalHandleError (err, vm, info) {
  if (config.errorHandler) {
    try {
      return config.errorHandler.call(null, err, vm, info)
    } catch (e) {
      logError(e, null, 'config.errorHandler')
    }
  }
  logError(err, vm, info)
}

function logError (err, vm, info) {
  if (process.env.NODE_ENV !== 'production') {
    warn(`Error in ${info}: "${err.toString()}"`, vm)
  }
  /* istanbul ignore else */
  if ((inBrowser || inWeex) && typeof console !== 'undefined') {
    console.error(err)
  } else {
    throw err
  }
}
```

- `cur = cur.$parent`从下往上，调用错误捕捉钩子`errorCapture`
- 如果`errorCapture`钩子返回`false`可以不用再调用`globalHandleError`，也代表错误已经被`errorCapture`处理了

### lang.js

**isReserved:**

```js
/**
 * Check if a string starts with $ or _
 */
export function isReserved (str: string): boolean {
  const c = (str + '').charCodeAt(0)
  return c === 0x24 || c === 0x5F
}
```

- 检查一个字符串是否含有`$`或者`_`
- 这两个字符代表是预留的变量如`$data`或`_data`这些都是预留的变量

**def:**

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

- 封装`Object.defineProperty`

**parsePath:**

```js
/**
 * Parse simple path.
 */
const bailRE = /[^\w.$]/
export function parsePath (path: string): any {
  if (bailRE.test(path)) {
    return
  }
  const segments = path.split('.')
  return function (obj) {
    for (let i = 0; i < segments.length; i++) {
      if (!obj) return
      obj = obj[segments[i]]
    }
    return obj
  }
}
```

- 缓存一个`a.b.c`的字符串，然后调用`obj.a.b.c`
- 比如有个对象`const obj = { a: { b: { c: 2 } } }`
- `parsePath('a.b.c')`后就返回一个函数`fn`，`fn(obj)`会直接返回`2`，也就是生成一个可以缓存要访问的对象的一系列键名的函数，
- 那个`path`应该很常用才使用，这样的做法可以避免一个变量是`undefined`的时候，还调属性`undefined.b`会报错，这种方式调用一个对象还可以避免繁琐的`obj && obj.a && obj.a.b && obj.a.b.c`
