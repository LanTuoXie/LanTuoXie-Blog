##wepy一些问题和解决方案

## computed属性名和props属性名重复

- 如果那个组件的渲染值是重名的computed属性，每次点击都是之前的计算加上最新计算的值
- 定义computed属性名的时候避免和props属性名重复

## 如何给当前组件或者页面类添加静态属性

- 可以在`constructor`构造函数中添加静态属性，但是要添加`super()`，才可以使用`this`来添加静态属性
- 由于`constructor`执行时间比较早，所有在这里面的变量，可以直接用于绑定模板，也就是可以和`data`的数据一样用于数据绑定
- 如果有些数据是一次性的，不会改变的（静态），最好在`constructor`中定义，不要都放到`data`中，`data`中的数据都会被转换成`Observable`数据的

## 组件与组件通信$broadcast、$emit、$invoke

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
- 应为它们共享同一个props，多个组件一起使用，类似定义多个同名的函数一样，最后使用的是最后一个函数
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
- 原生组件的情况下，可以使用`.`或者`[]`的方式选择想传的数据，但是自定义组件不可以，要整个数据传入，或者传入`item`(`[{}, {}]`这种格式的数据)，`item`就是`wx:item`
- 数据源要在`data`中，才有效果，静态属性不行
- 尽量只包含一个组件，出现多个组件，会出现莫名的bug，其它的功能再在这一个组件内部处理，然后组件间的通信通过`$broadcast、$emit、$invoke`
- 使用组件多使用`$broadcast、$emit、$invoke`的方式传值，别老想着使用`props`，会有很多莫名的bug
- 关于`$broadcast、$emit、$invoke`，可以通过`mixins`拯救一下，唯一没什么莫名bug的

## Object.create(null)

- 上面返回的对象不带原型链，也即不包含`Object`原生对象的信息
- 小程序中一些数据，像`data`需要的对象要带原型链，不能用这个,否则在绑定数据到模板的时候会报错
