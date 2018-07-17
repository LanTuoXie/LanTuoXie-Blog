## MobX(1)

`MobX`是一个简单和可扩展的状态管理。
![图片](https://cn.mobx.js.org/flow.png)

- `Actions`：类似Redux或者Vuex中的Actions可以异步和副作用的操作
- `State`：Redux和Vuex都是单状态树，但是MobX可以多个状态
- `Computed Values`：类似Vuex中的getter
- `Reactions`：类似Redux中的connect

#### 核心概念
- Observable State (可观察状态)
- Computed Values (计算值)
- Reactions (反应)
- Observer (组件订阅状态)

#### MobX基本操作和要点
- 定义状态并使其可观察
- 创建视图以响应状态的变化
- 更改状态
```js
import { observable, observer, action } from 'mobx'

// 定义状态并使其可观察
const appState = ovserable({
    timer: 0
})

// 定义actions 改变状态
appState.resetTimer = action(function reset() {
    appState.timer = 0
})

// 创建视图以响应状态的改变
@observer
class TimerView extends React.Component {
    onReset = () => this.props.appState.resetTimer()

    render() {
        return (
            <button onClick={this.onReset}>
                Seconds passed: { this.props.appState.timer }
            </button>
        )
    }
}

```

#### 概念与原则
- State：状态是数据，那个数据会随着时间迁移可能发生改变。也即不同时刻，数据可能拥有不同值（非固定、会变化、可修改的数据）。
- Derivations：基于某个公式和某个状态而衍生出来的值或者数据，`Computed Values` 和 `Reactions`。黄金法则：如果你想创建一个基于当前状态的值时，请使用`computed`
- Actions：任何一段可以改变状态的代码。

#### Observable State

Observable值可以是JS基本数据类型、引用类型、普通对象(不使用构造函数创建的对象如'{}')、类实例、数组和映射。
- `Observable Map`：如果值是ES6的`Map`
- `Observable Array`：如果值是数组
- `Observable Object`：如果值是没有原型的对象
- `Boxed Observable`：如果值是有原型的对象，或者JS原始数据，或者函数

如果要创建一个**键是动态的对象**，使用`Observable Map`，因为对象上只有在初始化时才会自动将对象上的属性转化成可观察的，动态添加的不会，但是可以通过extendObserable转化成可观察的

```js
// 创建 Observable Map
const map = observable.map({ key: 'LanTuoXie' })
map.set('key', 'MyDuty')

// 创建 Observable Array
const list = observable([1, 2, 3])
list[2] = 3

// 创建 Observable Object
const person = observable({
    firstName: 'Lan',
    lastName: 'TuoXie'
})
person.firstName = 'My'
person.lastName = 'Duty'

// 创建 Boxed Observable
const commonData = observable.box(20)
commonData.set(25)
```

#### Observable Object

创建`Observable Object`类型：`obserable.object(props, decorators?, options?)`

- 只有在把对象转变observable时存在的属性才是可观察的，动态添加的属性要使用`set`或`extendObserable`处理才可以让其可观察。
- 属性的getter会自动转变成衍生属性，也即如`@computed`的效果。
- observable会自动递归地处理整个对象
- 这里面的普通对象指的是不是使用构造函数创建出来的对象，或者没有原型的对象，简单点就是字面量的对象`{}`。

#### Observable Array

创建`Observable Array`类型：`observable.array(values?) 或 observable([])`   
是否深度递归添加观察：`observable.array(values, { deep: false })`

- 注意：`observable.array`返回的是一个类数组对象（人造数组）。`Array.isArray(observable([]))`返回`false`
- `observable([]).slice()`可以转为真正的数组
- observableArray的`sort`和`reverse`不会改变原来的数组，而是返回一个新的排过序的数组。
- 支持原生Array所有方法。

除了支持原生Array的所有方法还扩展了以下方法。

- `intercept(interceptor)`：钩子函数，当数组变化时，作用域数组前将其拦截
- `const unobserve = observe(listener, fireImmediately? = false)` 订阅数组的变化
- `clear()` 删除数组所有项
- `replace(newItems)` 用新项替换数组中的所有项
- `find(predicate: (item, index, array) => boolean, thisArg?)` ES7的`Array.find`
- `findIndex(predicate: (item, index, array) => boolean, thisArg?)` ES7的`Array.findIndex`
- `remove(value)` 通过值从数组中移除一个单个的项，被移除返回`true`。
- `peek()`和`slice()`类似，单`peek`不创建保护性拷贝。

#### Observable Maps

创建`Observable Maps`：`observable.map(values?) 或 observable(new Map()) 或 @observable map = new Map()`

- ES6 Map 支持的方法`Observable Maps`全支持，还扩展了自己的方法。
- 支持动态添加键，动态添加的会触发观察属性的观察效果。

ES6 Map原生支持的方法：

- `has(key)` ： 是否存在该项
- `set(key, value)` ： 给键设置值
- `delete(key)` ： 删除键
- `get(key)` ： 获取该键值
- `keys()` : 返回所有键名
- `values()` : 返回所有键值
- `entries()` ： 返回所有键值对的数组 `[[key1, value1], [key2, value2]]`
- `forEach(callback:(value, key, map) => void, thisArg?)` ： 遍历，注意第一个是参数是value，和`Array.prototype.forEach`的callback有点区别
- `clear()` ：全清空
- `size` ： 项的数量

MobX自己扩展的

- `toJS()` 转为原生JS的`Map`类型
- `toJSON` 返回此映射的浅式普通对象表示。
- `intercept(interceptor)` ： 变化前拦截
- `observe(linstener, fireImmediately?)` ：监听map的变化
- `merge(values)` ：合并，`values`可以是普通对象、enteries数组、ES6的Map
- `replace(values)` : 用提供值替换映射全部内容

#### Boxed Observable

`observable.box(value)`接收任何值并把值存储到箱子中。使用`get()`可以获取当前值，使用`set(newValue)`可以更新值。 

- `get`：返回当前值。
- `set(value)`：替换当前存储的值并通知所有观察者。
- `intercept(interceptor)`：变化前拦截。
- `observable(callback:(change) => void, fireImmediately) = false`：注册一个观察者

#### Decorators（装饰器）

`obserable(data, decorators?, options?)`的decorators    
主要有三部分`observable`、`computed`、`action`
- `observable`：修饰可观察属性是否深克隆、是否自动转换成可观察等
- `computed`：创建一个计算属性，以及一个怎样的计算属性
- `action`：创建一个动作，以及一个怎样的动作

**observable**

- `observable.deep 或 observable`：深度可观察性，深度递归地给属性添加观察属性。
- `observable.ref`：禁用自动的observable转换，只是创建一个observable引用（不观察该属性）。
- `observable.shallow`：不递归、只能与集合组合使用。将任何分配的集合转换为oibservable，但该集合的值将按原样处理。
- `observable.struct`：和`ref`效果一样，但会忽略结构上等于当前值得新值。

**computed**

- `computed(options)` ：设置选项
- `computed.struct` ：只有当前视图产生的值与之前的值结构上有不同时，才通知它的观察者

**action**

- `action(name)` ： 创建一个动作，重载名称
- `action.bound` ：创建一个动作，并将this绑定到了实例
