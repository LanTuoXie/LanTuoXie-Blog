## wepy框架构建小程序（1）

基本操作：

```bash
# 安装脚手架工具
npm install wepy-cli -g

# 创建一个新的项目
npm init standard myproject

# 进入新项目文件夹和安装依赖
cd myproject && npm install

# 跑项目
npm run dev
```

### vscode 编辑器设置

在编辑器设置中设置：

```json
{
  // 小程序设置为false
  "vetur.validation.template": false
}
```

由于编辑器的设置有分`用户设置`和`工作区设置`，`用户设置`就是全局的基本设置，由于上面的功能在`Vue项目`需要开启，所以上面的设置代码应该在`工作区设置中设置`。

简单点说就是，`用户设置`是全局设置，而`工作区设置就是当前打开的文件夹有效`，也就是局部的设置。

### 定义事件（重点）

事件在web端是很重要的部分，这里要搞得清清楚楚。   
事件分`捕获阶段`、`事件触发点`、`冒泡阶段`。   
我们用的最多的是`冒泡阶段`。原生小程序定义事件的方式是键值对的方式`key:value`。

原生小程序的`keys`:

- `bind`：冒泡阶段，对应web端的`on`
- `catch`：用这个key来定义事件不会冒泡，相当于web端的`event.stopPropagation()`
- `capture-bind`：这个key用来定义捕获阶段，不像web端，我们要兼容IE，只考虑冒泡，因为IE老版本没有捕获
- `capture-catch`：这个key在触发事件后，终止捕获，由于是`捕获 -> 触发事件 -> 冒泡`，所以终止捕获也终止了冒泡

`bindtap`：在小程序中用`tap`替代`click`，移动端用`tap`是避免点击事件的`300ms`延时造成的bug，`tap`触发更快。   
`bind:tap`：这个方式也可以，就是`key:value`方式定义事件，`value`就是`事件名`，其他事件名大多和web端一样

wepy的事件定义：

- `bindtap`替换`@tap`
- `catchtap`替换`@tap.stop`
- `capture-bind:tap`替换`@tap.capture`
- `capture-catch:tap`替换`@tap.capture.stop`

这些语法就是`Vue`的语法，就是要区分这个`key:value`，key是什么，value是什么

### 安装Sass

```bash
npm i wepy-compiler-sass --save-dev
```

然后在`<style lang='scss'>`

### `.wpy`文件

- `<template>`：对应`.wxml`
- `<script>`：分成两部分。1、逻辑部分：除了config对象，其他对应`.js`文件；2、配置部分：即config对象，对应`.json`
- `<style>`：对应`.wxss`
- 注意：由于`config`对象对应`.json`，但是可以是原生js对象，最后会格式化成JSON

上面3个标签都支持`src`和`lang`属性：

- `src`：引入外联的文件，使用了这属性，内联的代码无效
- `lang`：有多个值

各个标签的`lang`：

- style：默认值`css`、可能值`css、less、scss、stylus、postcss`
- template：默认值`wxml`、可能值`wxml、xml、pug`
- script：默认值`babel`、可能值`babel、TypeScript`

### 状态管理器： `redux` + `wepy-redux`

先用`redux`生成`store`

```js
import { createStore, combineReducers, applyMiddleware } from 'redux'
import promiseMiddleware from 'redux-promise'
import * as reducers from './reducer'

// 生成的store和React中的redux生成方法一模一样
const store = createStore(
  combineReducers(reducers),
  applyMiddleware(promiseMiddleware)
)
```

`React`注入`store`是通过`<Provider>`，而`wepy-redux`注入`store`的方法用`setStore(store)`。    
`wepy`有3中实例`App实例`、`Page实例`、`Component实例`，`App`只能有一个，注入`store`就是要在`app.wpy`中注入

```js
import { setStore } from 'wepy-redux'

// app.wyp
setStore(store)

```

- `setStore(store)`要在`wepy.app`的页面注入

然后就可以在任何一个组件中使用：

```js
  import { connect } from 'wepy-redux'

  @connect({
    num(state) {
      return state.counter.num
    },
    inc: 'inc'
  }, {
    addNum: INCREMENT,
    asyncInc
  })
```

- `connect(states, actions)`
- `states`：可以是数组`Array`或者对象`Object`，用法和`Vuex`的`mapState`一样

```js
// states为数组，元素只能是字符串，和`vuex`的用法一样
@connect(['num', 'inc'])

```

- `actions`：只能是对象`Object`，对象的`value`可以是字符串(`action.type`)或者函数

```js

@connect(null, {
  test: 'TEST',
  test2: (...args) => ({ type: 'TEST', payload: args }),
  test3: { type: 'TEST', payload: 1 }
})

// 对于value是字符串，`test(1)`相当于
dispatch({
  type: 'TEST',
  payload: 1
})

// 调用`test(1, 2)`相当于
dispatch({
  type: 'TEST',
  payload: [1, 2]
})

// 调用`tes2()`相当于
dispatch(val.apply(store, args)) // this 指向store 但是自定义的函数要返回符合 { type: 'TEST', payload }的`action`对象

// 调用`test3()`
dispatch({ type: 'TEST', payload: 1 })

```

- 最主要的是dispatch还是redux的dispatch，搞成这样是想把dispatch搞成和`vuex`的dispatch一样使用，可以支持异步（redux中原本的dispatch是同步的）
- 这里的dispatch是通过`middleware`修改了redux原生的dispatch，使用了`redux-promise`，这样`dispatch`就可以支持异步以及promise
- wepy还使用了`redux-actions`，来实现`mutations`和`actions`。如果不喜欢这个可以自己换`redux-thunk`，都可以，这些内容都属于redux（异步action），有自己的实现方案就行，你甚至都不用也可以。
- 关于actions，如果是异步的，用test2的方式，同步的直接一个type字符串或者一个action对象

### 页面生命周期钩子、组件生命周期钩子

- `onLoad`
- `onShow`
- `onReady`
- `onHide`
- `onUnload`

其中组件和页面公用`onLoad`和`onShow`

还要区分三大实例`App实例`、`Page实例`、`Component实例`

### 组件间通信

小程序的组件间通信和web端的组件是有分别的，小程序的组件，两者通信主要通过广播的方式。

- `$broadcast`：事件是由父组件发起，所有子组件都会收到此广播事件，除非事件被手动取消
- `$emit`：事件发起组件的所有祖先组件会依次接收到`$emit`事件
- `$invoke`：是一个页面或者组件对另一个组件中的方法的直接调用

```js

// 所有在当前页面或者组件中的组件，如果events定义了changeState事件，就会调用其回调函数，以及传入参数
this.$broadcast('changeState', arg1, arg2, ...)

// 所有在当前组件的父组件或者父页面中，如果events定义了getId事件，就会调用其回调函数，以及 传入参数11
this.$emit('getId', 11)

// 调用当前页面或者当前组件中的组件AddNumber的方法，方法add定义于methods中，以及传入参数2，相对于AddNumber.methods.add(2)
this.$invoke('AddNumber', 'add', 2)

```