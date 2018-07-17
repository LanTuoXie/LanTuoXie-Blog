## MobX(2)--Reaction

#### @computed

- 计算值是根据现有的状态或其它计算值衍生出来的值。
- 计算值是高度优化过的，尽可能的多使用它们。
- 计算属性是不可枚举的，它们也不能在继承链中被覆盖。
- 普通类中定义@computed要配合getter使用
- `observable.object`和`extendObservable`会自动将getter属性推导成计算属性，不用添加computed修饰符。
- 永远在getter之后定义setter
- setter不能用来直接改变计算属性的值，但他们可以用来做逆向衍生
- 要区分`@computed`和`autorun`

**computed和autorun的区别**

- computed是比较纯的，应该避免副作用，而autorun可以出现副作用。
- 如果一个计算值不再被观察，MobX可以自动地将其垃圾回收。而autorun要手动清理。

#### autorun

创建一个响应式函数，而该函数本身永远不会有观察者。主要用于打印日志、持久化或者更新UI的代码。`@observer(React Component)`就是用了`autorun`来更新组件。

- 当使用`autorun`时，所提供的函数总是立即被触发一次，然后每次它的依赖关系改变时会再次被触发。
- `computed(function)`创建的函数只有当它有自己的观察者时才会重写计算，否则它的值会被认为是不相关的。
- 规则：如果你有一个函数应该自动运行，但不会产生一个新的值，请使用`autorun`
- `const disposer = autorun(function, options)` 返回的`disposer函数`用于清除`autorun`

`autorun`的`options`

- `delay`：用于效果函数进行去抖，单位为毫秒ms
- `name`：利于调试
- `onError`：报错的时候的
- `scheduler`：设置自定义调度器以决定如何调度autorun函数的重新运行

#### when

使用：`when(predicate: () => boolean, effect?: () => void, options?)` 
when观察并运行给定的predicate，直到返回true.一旦返回true，给定的effect就会被执行，然后autorunner会被清理。该函数返回一个清理器以提前取消自动运行程序。

```js
class Resource {
    constructor() {
        when(
            () => !this.isVisible,
            () => this.dispose()
        )
    }

    @computed get isVisible() {
        // 标识此项是否可见
    }

    dispose() {
        // 清理操作
    }
}
```

如果没有提供`effect`函数，`when`会返回一个`Promise`。可以与`async / await`配合
```js
async function() {
    await when(() => that.isVisible)
}
```
#### reaction

用法：`reaction(() => data, (data, reaction) => { sideEffect }, options?)`。    

- 更加细颗粒的`autorun`
- 第一参数是数据函数(映射)，用来追踪并返回数据作为第二个函数的输入
- 不同于`autorun`的是，当创建时效果函数不会直接运行，只有数据表达式首次返回一个新值后才会运行。且在执行效果函数时访问的任何observable都不会被追踪
- reaction 返回一个清理函数。传入reaction的函数当调用时会接收两个参数，即当前的reaction。
```js
const counter = obaservable({ count: 0 })

const reaction = reaction(
    () => counter.count,
    (count, reaction) => {
        console.log('计数：', count)
        // 清除当前reaction
        reaction.dispose()
    }
)

counter.count = 1
// 输出：
// 计数：1

counter.count = 2
// 没有输出
```

#### @observer

`observer`可以将React组建转变成响应式组建。它用`mobx.autorun`包装了组件的`render`函数以确保任何组件渲染中使用的数据变化时都可以强制刷新组件。

需要`mobx-react`。
