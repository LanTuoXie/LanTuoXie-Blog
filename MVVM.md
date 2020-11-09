# Vue和MVVM的对应关系

Vue是受MVVM启发的，那么有哪些相同之处呢？以及对应关系？

## [MVVM(Model-view-viewmodel)](https://zh.wikipedia.org/wiki/MVVM)

MVVM还有一种模式**model-view-binder**，主要用来简化用户界面的**事件驱动程序设计**

MVVM可以分成四部分

- `Model:模型`
- `View:视图`
- `ViewModel:视图模型`
- `Binder:绑定器`

主要形式还是**Model-ViewModel-View**</br>
![模型图](https://upload.wikimedia.org/wikipedia/commons/thumb/8/87/MVVMPattern.png/330px-MVVMPattern.png)

>模型：是指代表真实状态内容的领域模型(面向对象)，或指代表内容的数据访问层(以数据为中心)</br>
>视图：是用户在屏幕上看到的结构、布局和外观(UI)</br>
>视图模型：暴露公共属性和命令的视图抽象。让视图和数据二者进行通信，靠的**绑定器** </br>
>绑定器：声明性数据和命令绑定

## Vue和这四部分的关系

对应关系：

- 视图：对应真实的html和css
- 视图模型：对应Vue的模板语法
- 绑定器：对应v-bind v-model @click :prop等绑定数据语法
- 模型：Vue的实例中的那些属性 $data $methods $computed 等等

在一个`.vue`文件中，我们会看到3部分`<template />` `<script />` `<style />`</br>
`<template />` 负责**视图模型和绑定器**</br>
`<style />` 负责**视图的CSS**</br>
`<script />` 中定义的Vue实例负责**模型的数据管理和绑定器的逻辑**

如何用Vue解释**Model-ViewModel-View**呢？

**ViewModel-View阶段**

视图模型转化为视图，也即Vue中的模板语法转化为实际的HTML和CSS，这个部分主要由Vue自动实现，我们开发者主要处理的是**Model-ViewModel**阶段。

**Model-ViewModel阶段**

这个阶段就是我们实例化Vue对象，添加data,methods等操作，以及将数据绑定到模板上。

```html
<template>
  <div class='test' @click='add'>{{count}}</div>
</template>
```

```js
// <script>
export default {
  data () {
    return {
      count: 0
    }
  },
  methods: {
    add (e) {
      this.count += 1
    }
  }
}
```

Model:定义data函数管理数据count,以及定义add函数控制count数据的变更</br>
ViewModel:是抽象语法，主要是Vue的模板语法，绑定数据，之后Vue会将其转化为真实的HTML</br>

由于，ViewModel和Model主要是**绑定关系**，也即是数据和视图是绑定的，你什么样的数据就决定了什么样的视图，所以我们一般也把Vue称为数据驱动框架。</br>
所以很多时候，只要知道数据和ViewModel的关系，也就知道UI的样子了，这个时候，我们只需操作数据的结构，也就操作了视图。

```html
<template>
  <ul class='list'>
    <li class='item' v-for='(v, index) in arr' :key='index'>{{v}}</li>
  </ul>
</template>
```

```js
export default {
  data () {
    return {
      arr: [1, 2, 3, 4, 5]
    }
  },
  created () {
    // 改变数据arr的数据结构，添加新的数值
    this.arr.push(6)
  }
}
```

Model和ViewModel的关系：

arr和`<li>`标签绑定，有多少个arr元素就有多少个`<li>`</br>
后面arr添加了一个元素`6`，这时候arr的长度是6，那应该有6个`<li>`，这就是数据和视图的绑定，由数据的结构我们就可以推出视图的样子。</br>
所以我们要多从操作数据的角度思考问题，当然前提是你已经确定了Model和ViewModel的绑定关系是怎样的。这个时候我们只需操作Model就可以了。</br>

上面的例子采用的数据结构是数组，当然还有很多数据结构。Model和ViewModel绑定后，基本上Model的数据结构就决定了。那么这时，我们只需根据这个Model的数据结构增删修改。</br>
还有一点就是vue中有多种绑定方式，v-if v-for 等等，一个ViewModel只有一个Model的数据结构，但是相同的View可以有多种ViewModel</br>
比如这个View`<div>hello</div>`，有多种ViewModel都可以生成这个，有多种ViewModel也即有多种Model(数据结构)

```html
<template>
  <div>{{data}}</div>
  <div>{{obj.data}}</div>
  <div>{{arr[0]}}</div>
</template>
```

```js
export default {
  data () {
    return {
      data: 'hello',
      obj: {
        data: 'hello'
      },
      arr: ['hello']
    }
  }
}
```

上面有3种ViewModel和3种Model 但生成的View都是一样的`<div>hello</div>`
