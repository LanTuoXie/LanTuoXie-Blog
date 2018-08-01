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

