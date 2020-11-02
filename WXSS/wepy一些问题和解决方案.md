# wepy一些问题和解决方案

小程序开发和传统的web开发有相识的地方，但是也有不同的地方，要区分。

## computed属性名和props属性名重复

- 如果那个组件的渲染值是重名的computed属性，每次点击都是之前的计算加上最新计算的值
- 定义computed属性名的时候避免和props属性名重复

## 父子组件传值和Vue中的区别

- Vue中的prop传值默认是只要父组件的数据改变，子组件同时改变，但是这种模式在wepy中是静态的 也即 `:name="name"`是静态的，父组件改变`name`子组件还是不会变，子组件还是父组件第一次传进来的值
- wepy要做到父组件数据变的同时，子组件也跟着变，要加个修饰符`.sync`，也即`:name.sync="name"`
- 还有一个是双向绑定的问题，Vue中可以通过`$emit`事件的方式实现，这里wepy可以在子组件中的`props配置类型、默认值、以及是否双向`中的`twoWay`为`true`，就实现了双向数据流
- 默认`twoWay`的值是`false`，如果不是特别需要，请按单向数据的方式传递数据最好，再配合`$emit、$broadcast、$invoke`，尽量少用，除非是表单这种

## 如何给当前自定义组件或者页面类添加静态属性

- 可以在`constructor`构造函数中添加静态属性，但是要添加`super()`，才可以使用`this`来添加静态属性
- 由于`constructor`执行时间比较早，所有在这里面的变量，可以直接用于wepy组件，也就是可以和`data`的数据一样用于数据绑定，但是原生组件只能放在`data`中
- 如果有些数据是一次性的，不会改变的（静态），最好在`constructor`中定义，不要都放到`data`中，`data`中的数据都会被转换成`Observable`数据的

## 组件与组件通信`$broadcast、$emit、$invoke`

- 这些方法不可以在`constructor`调用，可以在`onLoad`中调用
- `events`用于定义事件，用于接收用的
- 如果想传数据给子组件，可以使用`$broadcast('eventName', ...args)`，但是所有的`后代`组件都会触发该事件，如果后代组件的`events.eventName`有定义到，会调用其函数，并将`args`传给其函数
- 如果想传数据给父组件可以使用`$emit`，同理，父组件（到根组件）的`events`要事先定义好事件
- `$broadcast和$emit`都会一次触发多个事件，定义`events`的时候要注意
- `$invoke('./componentName', 'methodName', args)`可以直接调用指定组件的方法`methods`中定义的方法，如果是调用父组件可以用`../`往上层走，如果要调用下层的`./oneLevelComponentName/twoLevelComponentName`
- 如果有很多组件都公用一个方法`methods`或者`events`，还可以使用`mixins`

## 区分Page、App、Component、Mixin这些实例

- 有些生命周期钩子是那个实例特有的，有些生命周期是所有都共享的

## components不同于Vue或React的组件

- 小程序的组件和Vue和React组件是有区别的
- 在wepy中引入组件在`components`中，和Vue的一样，但是使用的时候是不同的，一个组件对应一个id，如果在同一个模板中使用同一组件超过2次，它们会共享数据
- 共享数据指的是props等，可以测试一下，同一个组件传入的props名一样，值不一样，最终所有组件那的那个props值是最后一个值
- 因为它们共享同一个props，多个组件一起使用，类似定义多个同名的函数一样，最后使用的是最后一个函数
- 为了避免这种情况下可以用同一个组件去定义多个组件名`{ coma: Button, comb: Button }`，实际上就是new多一个`Button`
- 也就是说小程序中的组件在使用的时候，不会自动地new一个新的组件，而是同一个组件，而像`Vue和React`这样的组件，它们会new一个新的`Button`实例
- 还有一种情况是，在循环体`<repeat>`中，在小程序中就会去new一个新的组件实例，如果你的内容重复，又不想定义多个组件名（new 多个组件），可以使用循环体`<repeat>`
- 还有一种情况是`slot`，如果那个组件和数据无关，只是单纯的用于`slot`模板，它们最终生成的标签是不同的，因为这些生成的模板和数据无关，如果生成的模板中包含数据，那生成那部分关联数据的元素都是一样的
- 这里说的数据是`<view class='{{ className }}' >{{ data }}</view>`这些花括号中的数据`className`和`data`，它们是共享的，所以它们最终生成的视图是一模一样的
- 小程序的组件和目前流行的web组件有很大的区别，不能老用web组件传值得思维思考（很多莫名的bug），多从广播（订阅）的角度思考

## repeat注意点

- 分两种情况

```html
//repeat包着元素view text等
<repeat for="{{mylist}}">
  <view>{{item.name}}</view>
</repeat>

//repeat包着自定义组件
<repeat for='{{myList}}'>
  <List :mylist.sync="mylist"></List>
  <Item :item='item'></Item>
</repeat>
```

- 不支持在`repeat的组件`中使用`props`、`computed`、`watch`等特性
- 在使用`repeat`的时候，尽量使用纯文本渲染的组件，纯组件
- 原生组件的情况下，可以使用`.`或者`[]`的方式选择想传的数据，但是自定义组件不可以，要整个数据传入，或者传入`item`(`[{}, {}]`这种格式的数据)，`item`就是`wx:item`
- 数据源要在`data`中，才有效果，静态属性不行
- 尽量只包含一个组件，出现多个组件，会出现莫名的bug，其它的功能再在这一个组件内部处理，然后组件间的通信通过`$broadcast、$emit、$invoke`
- 使用组件多使用`$broadcast、$emit、$invoke`的方式传值，别老想着使用`props`，会有很多莫名的bug
- 关于`$broadcast、$emit、$invoke`，可以通过`mixins`拯救一下，唯一没什么莫名bug的

## Object.create(null)

- 上面返回的对象不带原型链，也即不包含`Object`原生对象的信息
- 小程序中一些数据，像`data`需要的对象要带原型链，不能用这个,否则在绑定数据到模板的时候会报错

## 字符串双引号""和单引号''

- 在模板中务必使用`""`，别用`''`（想杀人）
- `:age='2'`和`:age="2"`，前面的`''`是传不了`2`给`age`的，除非使用`"2"`，哈哈哈哈哈哈哈哈
- 在模板中最外层字符串最好用`""`不要使用`''`，比如定义事件函数的时候`'handlerTap("123")'`，这里是不成功的给`handlerTap`传递参数`"123"`，但是`"handlerTap('123')"`就可以???
- 正确用法：`@tap="handlerTap('love')"`，最外层用`""`，最内层`''`

## mixins的computed顺序问题

- mixins中的computed执行顺序和vue的相反，也即先执行component或者页面的定义的，然后再执行mixins中的
- 还有就是延迟的问题，mixins的慢于组件或者当前页面定义的，如果想在当前页面获取mixins的一些值，会获取不到

## 组件传值和模板绑定问题

- 一个是上面提到的字符串要用双引号
- 然后就是要区分原生组件和自定义组件

原生组件：就是小程序自带的组件`<view>、<text>、<image>等等`，这些原生组件绑定数据，要求数据必须在`data`中，如果绑定的数据不在`data`中将不会渲染

```vue
<template>
  <view class="{{className}}">{{text}}</view>
</template>

<script>
import wepy from 'wepy'

// 注意：这个是组件实例
export default class Test extends wepy.component {
  data = {
    text: 123,
    className: 'hahah'
  }
  
  // 由于computed最后返回值会成为data中一部分，computed也可以用于原生组件数据绑定
  // 但是在repeat的时候，不可以使用这个，repeat的时候不可以使用computed和watch
  computed = {
    text () {
      return this.text + 1
    }
  }
}
</script>
```

自定义组件：绑定数据可以是`data`中，也可以用自定义函数，或者静态变量来绑定数据，且绑定数据的方式要和原生小程序的一样，不能用vue的方式

```vue
<template>
  <UserComponent :fn="fn()" :staticV="staticVar" :text="text"/>
</template>

<script>
import wepy from 'wepy'

// 注意：这个是页面实例
export default class Index extends wepy.page {
  constructor () {
    super()
    this.staticVar = 123
  }

  data = {
    text: 123
  }
  
  // 自定义函数
  fn () {
    return '123'
  }
  
  // 由于computed最后返回值会成为data中一部分，computed也可以用于原生组件数据绑定，如果和data中的属性重名，优先使用computed的
  computed = {
    text () {
      return this.text + 1
    }
  }
}
</script>
```

原生组件的绑定只能是data中的数据，不可以像vue中的那样绑定数据，但是如果是自定义的组件就可以像vue那样绑定数据。还有`methods`自能用于定义事件处理程序，不能把自定义函数定义在里面

也就是碰到原生组件的时候，请用原生小程序的方式处理，碰到自定义组件的时候才考虑用你vue的方式处理，也就是区分原生组件和自定义组件非常重要

### 图片和导航的路径问题

- 不要在组件中使用相对定位，如`../`这种，因为当组件被引入到某个页面时，会相对哪个页面，导致路径无法复用
- 使用绝对定位，`/`，在小程序中，`/`是指的当前项目的文件夹，例如：`/pages/index`，这里的首个字符`/`就相当于`@/`都是指向`/src`目录
- 如果是图片，如`@/assets/icons/logo.png`可以用`/assets/icons/logo.png`，因为第一个字符是`/`就相当于`@/`，表示`/src`目录之下
- 如果是导航`wx.navigateTo`，如果导航到`/pages/user`，不要使用`/`或者`../`，直接`user`，最终会自动拼成`/pages/user`，也就是说微信默认`/pages/`+`user`

### canvas中的canvas-id不能动态赋值

- 可以通过slot来实现一个组件，不同canvas-id

```js
  <component-name sid="canvas">
    <canvas canvas-id="canvas"></canvas>
  </component-name>
```

- `component-name`是一个组件，接收sid只要是为了可以`wx.createCanvasContext(this.sid)`
- 其实我们在组件的内部只需要知道canvas的id，并且调用`wx.createCanvasContext(this.sid)`，所以我们可以把canvas独立成slot

### 在一个组件中引入另一个组件作为子组件，子组件的样式`<style>`不起作用

- 是因为父组件没有`<style>`标签，导致子组件不编译`<style>`标签的内容
- 最好还是每个组件都添加`<style>`标签，即使里面没有内容

### props不推荐传函数

在父组件和子组件间传递信息可以通过传递一个`this`绑定了父组件的函数给子组件，作为`$emit`的时候使用，但是在wepy中，只要是用于绑定模板的数据都会转为`data`的数据    
但是`data`中wepy是不推荐定义函数的，会报警告。

如何解决这个父组件传递子组件？比如现有一个`BarInput`的`input`组件，父组件要监听这个组件的`oninput和onconfirm`事件，可以这样

```js
// 父组件
{
  evnets: {
    'bar-input_oninput' (value) {
      // todo
    },
    'bar-input_onconfirm' (value) {
      // todo
    }
  }
}
```

```js
// BarInput 组件
{
  methods: {
    onInput(e) {
      const value = e.detail.value
      this.$emit('bar-input_oninput', value)
    },
    onConfirm(e) {
      const value = e.detail.value
      this.$emit('bar-input_onconfirm', value)
    }
  } 
}
```

注意上面`events`中的`'bar-input_oninput'`，先以组件名称起头，最后以事件类型结尾，借用参考`BEM`的规范，定义事件名很重要，因为`$emit`是向上一直触发`'bar-input_oninput'`事件，直到最顶端，如果有些事件名称比较简单，几个组件的事件名重名了，会触发该事件，一个唯一又独特的事件名就很有必要了。

父组件传递给子组件可以通过`$broadcast`

### camera组件

- 由于`camera`组件都是小程序中比较特殊的组件，要想使用`z-index`是没什么效果的
- 要使用`cover-view`组件
- 使用了`position: absolute`的情况下，这个组件使用`z-index`没效果，在开发中看到可以，但是在真机中不行
- 如果标签的排列顺序如果在下方，会是最高的z-index，前提是这个组件`position: absolute`设置了这个定位
- 根据偏移量设置有偏差，特别是底部`bottom: 0`是不会到底部，可以`bottom: -0.1%`，这个组件的定位有偏差，需要`0.1%`
- 需要根据`position: absolute`定位做一种照相聚焦的效果，这时候要用到`cover-view`，可以做到，但是由于定位有点偏差，需要调调，否则会有一条小黑线
- 小黑线是定位有误差，2个遮罩板重叠了
