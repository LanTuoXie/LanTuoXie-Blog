# Vue源码分析(1)

源码很多用到Flow
因为Vue涉及到多个平台以及服务端渲染，这里只分析Web端的。

- web
- weex
- server

先简后难，先从工具库入手

## ./src/shared

这个文件夹的文件是所有都共享的，比较简单，先从这里面入手

### ./src/shared/constants.js

```js
export const SSR_ATTR = 'data-server-rendered'

export const ASSET_TYPES = [
  'component',
  'directive',
  'filter'
]

export const LIFECYCLE_HOOKS = [
  'beforeCreate',
  'created',
  'beforeMount',
  'mounted',
  'beforeUpdate',
  'updated',
  'beforeDestroy',
  'destroyed',
  'activated',
  'deactivated',
  'errorCaptured'
]
```

- 全局常量库
- 所有生命周期的钩子以及资源类型
- 注意加多了三个生命周期钩子`activated`、`deactivated`、`errorCaptured`

### ./src/shared/util.js

一些常用且很有用的工具函数

> 注意：很多地方使用flow

**emptyObject:**

```js
export const emptyObject = Object.freeze({})
```

- 空对象，`Object.freeze()`只能查看，不可以修改
- 还有一种是`Object.create(null)`

**isUndef、isDef、isTrue、isFalse:**

```js
// these helpers produces better vm code in JS engines due to their
// explicitness and function inlining
export function isUndef (v: any): boolean %checks {
  return v === undefined || v === null
}

export function isDef (v: any): boolean %checks {
  return v !== undefined && v !== null
}

export function isTrue (v: any): boolean %checks {
  return v === true
}

export function isFalse (v: any): boolean %checks {
  return v === false
}
```

- `isUndef`是否未定义，这里未定义是数据类型为`undefined`或者`null`
- `isDef`是否定义
- `isTrue、isFalse`主要判断是不是Boolean值且是`true`还是`false`。有种情况是`'true'`、`'false'`它们转换成布尔值都是`true`

**isPrimitive:**

```js
/**
 * Check if value is primitive
 */
export function isPrimitive (value: any): boolean %checks {
  return (
    typeof value === 'string' ||
    typeof value === 'number' ||
    // $flow-disable-line
    typeof value === 'symbol' ||
    typeof value === 'boolean'
  )
}
```

- 判断是不是基本类型值`String`、`Number`、`Boolean`还有ES6的`Symbol`

**isObject:**

```js
/**
 * Quick object check - this is primarily used to tell
 * Objects from primitive values when we know the value
 * is a JSON-compliant type.
 */
export function isObject (obj: mixed): boolean %checks {
  return obj !== null && typeof obj === 'object'
}
```

- 很参见的判断是不是对象，先判断是不是`null`然后再根据`typeof`判断是不是对象

**_toString、toRawType:**

```js
/**
 * Get the raw type string of a value e.g. [object Object]
 */
const _toString = Object.prototype.toString

export function toRawType (value: any): string {
  return _toString.call(value).slice(8, -1)
}
```

- 根据`toString`返回的`'[object Type]'`来判断`引用类型`，是比较可靠的，很多库都用到
- `toRawType`返回的是`'[object Object]'`的`'Object'`

**isPlainObject、isRegExp:**

```js
/**
 * Strict object type check. Only returns true
 * for plain JavaScript objects.
 */
export function isPlainObject (obj: any): boolean {
  return _toString.call(obj) === '[object Object]'
}

export function isRegExp (v: any): boolean {
  return _toString.call(v) === '[object RegExp]'
}
```

- `isPlainObject`：通过`toString`严格判断是不是对象`Object`
- `isRegExp`：通过`toString`严格判断是不是`RegExp`

**isValidArrayIndex:**

```js
/**
 * Check if val is a valid array index.
 */
export function isValidArrayIndex (val: any): boolean {
  const n = parseFloat(String(val))
  return n >= 0 && Math.floor(n) === n && isFinite(val)
}
```

- 是不是合法的数组索引
- 索引要求是大于0的正整数且不能超过有限值，所以要通过`isFinite`来判断是不是有限值

**toString:**

```js
/**
 * Convert a value to a string that is actually rendered.
 */
export function toString (val: any): string {
  return val == null
    ? ''
    : typeof val === 'object'
      ? JSON.stringify(val, null, 2)
      : String(val)
}
```

- 将`null`转为`''`
- 将引用类型转为字符串，用的JSON.stringify，`JSON.stringify(val, null, 2)`中的2代表字段名的缩进，缩进2个space
- 基本类型值直接`String(val)`

**toNumber:**

```js
/**
 * Convert a input value to a number for persistence.
 * If the conversion fails, return original string.
 */
export function toNumber (val: string): number | string {
  const n = parseFloat(val)
  return isNaN(n) ? val : n
}
```

- 用于`<input />`的输入值，一般为字符串
- `parseFloat(val)`可以将符合数字类型规范的字符串转为浮点数，如果不符合解析为NaN
- `isNaN(n)`判断是否转化成功，成功用转化后的值（已经转为Number类型了），失败直接返回原来的值
- 这个技巧用于判断一个字符串是否符合数字类型规范很有用，或者这个字符串能否转为Number类型

**makeMap:**

```js
/**
 * Make a map and return a function for checking if a key
 * is in that map.
 */
export function makeMap (
  str: string,
  expectsLowerCase?: boolean
): (key: string) => true | void {
  const map = Object.create(null)
  const list: Array<string> = str.split(',')
  for (let i = 0; i < list.length; i++) {
    map[list[i]] = true
  }
  return expectsLowerCase
    ? val => map[val.toLowerCase()]
    : val => map[val]
}
```

- `Object.create(null)`会创建一个空对象`{}`不带原型链
- 这里只要是创建一个函数，一个缓存了一些key的map（map存于闭包中），这个函数主要功能是判断`传入的key`时候存在于`闭包中的map的key`
- 主要创建这个判断函数`key => map(key)`，map存于闭包中，且map中的key由`str`来生成，如果`str`是`a,b,c,d`就会创建这样的map`{a: true, b: true, c: true, d: true}`
- `expectsLowerCase`主要用于生成的map的key是否区分大小写，为真则不区分大小写，为假则区分（不传参数，默认值）

**isBuiltInTag、isReservedAttribute:**

```js
/**
 * Check if a tag is a built-in tag.
 */
export const isBuiltInTag = makeMap('slot,component', true)

/**
 * Check if a attribute is a reserved attribute.
 */
export const isReservedAttribute = makeMap('key,ref,slot,slot-scope,is')
```

- `isBuiltInTag`主要判断当前的HTML标签有`slot || component`
- `isReservedAttribute` 主要判断当前HTML标签的属性是不是预设值`key || ref || slot || slot-scope || is`
- 两个判断函数都是，都是与操作，只要符合任何一个key都返回true
- 也可以知道`key、ref、slot、slot-scope、is`是预设值，也即设置了这些属性的值，就有预设的功能，一共5个预设值属性，笔记

**remove:**

```js
/**
 * Remove an item from an array
 */
export function remove (arr: Array<any>, item: any): Array<any> | void {
  if (arr.length) {
    const index = arr.indexOf(item)
    if (index > -1) {
      return arr.splice(index, 1)
    }
  }
}
```

- 就是封装了Array.prototype.splice的移除操作
- splice会改变原来的数组，且返回移除的那个元素
- 这里还判断了数组的长度，长度为0的返回undefined

**hasOwnProperty:**

```js
/**
 * Check whether the object has the property.
 */
const hasOwnProperty = Object.prototype.hasOwnProperty
export function hasOwn (obj: Object | Array<*>, key: string): boolean {
  return hasOwnProperty.call(obj, key)
}
```

- 封装了Object.hasOwnProperty方法，用于判断一个对象或者数组是否存在一个属性
- hasOwnProperty是判断对象实例的可遍历的属性，不判断原型链的，也用于过滤原型链的属性
- `[].hasOwnProperty('slice')`是返回`false`的，`slice`是在原型链中的

**cached:**

```js
/**
 * Create a cached version of a pure function.
 */
export function cached<F: Function> (fn: F): F {
  const cache = Object.create(null)
  return (function cachedFn (str: string) {
    const hit = cache[str]
    return hit || (cache[str] = fn(str))
  }: any)
}
```

- 有用且简单的缓存方式，还是利用闭包来实现缓存。`cache`的存储结构是哈希表的方式
- 闭包中缓存了2个值：cache和fn，最终生成的函数`return hit || (cache[str] = fn(str))`，先看是否有缓存，没有再跑`cache[str] = fn(str)`，最终返回的还是缓存结构中的
- 主要功能是：缓存一个函数`fn`的运行结果，下次跑的时候，如果有缓存就不用再执行一次`fn`，直接取缓存的结果
- 注意：这个函数没有设计缓存区大小限制，还有`fn`要求是纯函数

**camelizeRE、capitalize、hyphenateRE:**

```js
/**
 * Camelize a hyphen-delimited string.
 */
const camelizeRE = /-(\w)/g
export const camelize = cached((str: string): string => {
  return str.replace(camelizeRE, (_, c) => c ? c.toUpperCase() : '')
})

/**
 * Capitalize a string.
 */
export const capitalize = cached((str: string): string => {
  return str.charAt(0).toUpperCase() + str.slice(1)
})

/**
 * Hyphenate a camelCase string.
 */
const hyphenateRE = /\B([A-Z])/g
export const hyphenate = cached((str: string): string => {
  return str.replace(hyphenateRE, '-$1').toLowerCase()
})
```

- `camelize`：是将由连字风格的字符串转为驼峰风格的若`my-name`转为`myName`。就是把`-n`转为`N`
- `return str.replace(camelizeRE, (_, c) => c ? c.toUpperCase() : '')`：第二个参数是个函数`_`是`match`，后面的`c`是`$1`就是捕获到的值
- `capitalize`：将字符串第一个字符转为大写
- `hyphenateRE`正好和`camelize`相反，是把`N`转为`-n`

**polyfillBind、nativeBind、bind:**

```js
/**
 * Simple bind polyfill for environments that do not support it... e.g.
 * PhantomJS 1.x. Technically we don't need this anymore since native bind is
 * now more performant in most browsers, but removing it would be breaking for
 * code that was able to run in PhantomJS 1.x, so this must be kept for
 * backwards compatibility.
 */

/* istanbul ignore next */
function polyfillBind (fn: Function, ctx: Object): Function {
  function boundFn (a) {
    const l = arguments.length
    return l
      ? l > 1
        ? fn.apply(ctx, arguments)
        : fn.call(ctx, a)
      : fn.call(ctx)
  }

  boundFn._length = fn.length
  return boundFn
}

function nativeBind (fn: Function, ctx: Object): Function {
  return fn.bind(ctx)
}

export const bind = Function.prototype.bind
  ? nativeBind
  : polyfillBind
```

- `polyfillBind`：向后兼容低级浏览器，给原生对象扩展bind函数。但是这个函数只是单纯的绑定，没有柯里化的效果`fn.bind(this, arg1, arg2, arg3)`，也即没有能缓存传入的参数的效果
- `nativeBind`：原生的bind函数，也是单纯的绑定this，不处理柯里化
- `bind`：就是`polyfillBind`和`nativeBind`的合体，适配模式，向下兼容

**toArray:**

```js
/**
 * Convert an Array-like object to a real Array.
 */
export function toArray (list: any, start?: number): Array<any> {
  start = start || 0
  let i = list.length - start
  const ret: Array<any> = new Array(i)
  while (i--) {
    ret[i] = list[i + start]
  }
  return ret
}
```

- 就是`Array.from`，将类数组的对象转为真数组，但是这个支持索引切割的方式转化，只有一个切割浮标

**extend:**

```js
/**
 * Mix properties into target object.
 */
export function extend (to: Object, _from: ?Object): Object {
  for (const key in _from) {
    to[key] = _from[key]
  }
  return to
}
```

- 将一个对象的属性混合到目标对象上，如果属性是引用类型，修改原来的对象，会影响混合后的对象

**toObject:**

```js
/**
 * Merge an Array of Objects into a single Object.
 */
export function toObject (arr: Array<any>): Object {
  const res = {}
  for (let i = 0; i < arr.length; i++) {
    if (arr[i]) {
      extend(res, arr[i])
    }
  }
  return res
}
```

- 讲所有元素都是对象的数组中的元素从左到右，开始混合成一个对象
- `[{ name: 'a' }, { age: 12 }, { spc: '帅' }, { name: 'MyDuty' }]` 变成`{ name: 'MyDuty', age: 12, spc: '帅' }`

**noop、no、identity:**

```js
/**
 * Perform no operation.
 * Stubbing args to make Flow happy without leaving useless transpiled code
 * with ...rest (https://flow.org/blog/2017/05/07/Strict-Function-Call-Arity/)
 */
export function noop (a?: any, b?: any, c?: any) {}

/**
 * Always return false.
 */
export const no = (a?: any, b?: any, c?: any) => false

/**
 * Return same value
 */
export const identity = (_: any) => _
```

- `noop`空函数，不执行任何操作，用于比如`[1, 2, 3].map(noop)`就可以返回一个都是undefined的数组`[undefined, undefined, undefined]`
- `no`是一个一定返回`false`的函数
- `identity`是一个输入什么就返回什么的函数，用于验证是否纯函数，还可以`[1, 2, 3].map(identity)`就可以克隆一个一模一样的数组
- 这个三个函数用于函数式编程中很有用，多留意

**genStaticKeys:**

由于vue是使用的flow类型系统，所有自定义的类型在`./flow`文件夹，这个函数用到`./flow/compiler.js`中的`ModuleOptions`类型

```js
declare type ModuleOptions = {
  // transform an AST node before any attributes are processed
  // returning an ASTElement from pre/transforms replaces the element
  preTransformNode: (el: ASTElement) => ?ASTElement;
  // transform an AST node after built-ins like v-if, v-for are processed
  transformNode: (el: ASTElement) => ?ASTElement;
  // transform an AST node after its children have been processed
  // cannot return replacement in postTransform because tree is already finalized
  postTransformNode: (el: ASTElement) => void;
  genData: (el: ASTElement) => string; // generate extra data string for an element
  transformCode?: (el: ASTElement, code: string) => string; // further transform generated code for an element
  staticKeys?: Array<string>; // AST properties to be considered static
};
```

```js
/**
 * Generate a static keys string from compiler modules.
 */
export function genStaticKeys (modules: Array<ModuleOptions>): string {
  return modules.reduce((keys, m) => {
    return keys.concat(m.staticKeys || [])
  }, []).join(',')
}
```

- 创建一个静态的`keys`字符串，这些key就是AST的属性
- 主要是处理模块的`staticKeys`，而`staticKeys?: Array<string>; // AST properties to be considered static`
- 合并所有模块中的`staticKey`成一个数组，然后用`','`分隔开

**looseEqual:**

```js
/**
 * Check if two values are loosely equal - that is,
 * if they are plain objects, do they have the same shape?
 */
export function looseEqual (a: any, b: any): boolean {
  if (a === b) return true
  const isObjectA = isObject(a)
  const isObjectB = isObject(b)
  if (isObjectA && isObjectB) {
    try {
      const isArrayA = Array.isArray(a)
      const isArrayB = Array.isArray(b)
      if (isArrayA && isArrayB) {
        return a.length === b.length && a.every((e, i) => {
          return looseEqual(e, b[i])
        })
      } else if (!isArrayA && !isArrayB) {
        const keysA = Object.keys(a)
        const keysB = Object.keys(b)
        return keysA.length === keysB.length && keysA.every(key => {
          return looseEqual(a[key], b[key])
        })
      } else {
        /* istanbul ignore next */
        return false
      }
    } catch (e) {
      /* istanbul ignore next */
      return false
    }
  } else if (!isObjectA && !isObjectB) {
    return String(a) === String(b)
  } else {
    return false
  }
}
```

- 两者都是基本类型之直接用`===`即可
- 还有一种情况是特殊值`NaN`是`===`比较不了的用`String`也即判断语句的`!isObjectA && !isObjectB`
- 一个基本一个引用直接返回`false`
- 两者都是引用类型要分数组和对象，如果一个数组一个对象直接返回`false`
- 都是数组：先比较数组的长度是否相等，里面元素的比较要递归该方法比较，注意这里使用`every`来遍历，遇到不相等的终止遍历，返回`false`都相等才返回`true`
- 都是对象：比较两个对象的键值，不等才遍历递归每个键的值，这里用了Object.keys，这个不包含原型链和Symbol，且要求是可遍历的属性，所以是浅相等判断
- 最主要的是：如何判断两个数组是否相等和两个对象是否相等，值得注意的是`NaN === NaN`，优先判断引用类型的结构是否相等，不等才深入判断值是否相等

**looseIndexOf:**

```js
export function looseIndexOf (arr: Array<mixed>, val: mixed): number {
  for (let i = 0; i < arr.length; i++) {
    if (looseEqual(arr[i], val)) return i
  }
  return -1
}
```

- 使用`looseEqual`来比较相等，返回数组的index也即是`const arr = [{}, undefined, { name: 'a' }]`使用`looseIndexOf(arr, { name: 'a' })`返回`2`

**once:**

```js
/**
 * Ensure a function is called only once.
 */
export function once (fn: Function): Function {
  let called = false
  return function () {
    if (!called) {
      called = true
      fn.apply(this, arguments)
    }
  }
}
```

- 还是用闭包缓存状态，记录是否被调用过，调用过了，就不再执行该函数，也就是说once返回的那个函数只能被调用一次