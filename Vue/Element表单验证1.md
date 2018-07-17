## Element表单验证（1）

> 首先要掌握Element官方那几个表单验证的例子，然后才看下面的教程。

Element主要使用了[async-validator](https://github.com/yiminghe/async-validator#length)这个库作为表单验证

`async-validator`主要分成三部分
- Validate
- Options
- Rules

其中，对于我们使用Element的来说，Rules最重要，也是这部分内容较多的。

#### async-validator各部分

**async-validator基本使用**
```js
import Validator from 'async-validator'

// 规则的描述
const rules = {
  name: { type: 'string', required: true }
}

// 根据规则生成验证器
const validator = new Validator(rules)

// 要验证的数据源
const source = {
  name: 'LanTuoXie'
}

// 验证后的回调函数
function callback (errors, fileds) {
  if (errors) {
    // 验证不通过，errors是一个数组，记录那些不通过的错误信息
    // fileds是所有数据源的字段名，也即上面的source的'name'
    // 验证是根据字段名来的，rules.name 对应 source.name。 字段名要一样才会验证
  }
  // 下面是验证通过的逻辑
}

// 验证数据源是否符合规则
validator.validate(source, callback)
```

**Validate**

就是上面例子中的validator.validate，是一个函数
```js
function(source, [options], callback)
```
source和callback对应上面的例子。

**Options**

Options有两个值
- `first`: 一个布尔值，如果出现字段不通过，终止验证后面的字段
- `firstFields`: 布尔值或者字符串，如果验证一个字段时，一个规则不通过，终止验证下个规则(一个字段，多个规则的情况)

`firstFields`是针对单个字段多规则的情况下使用，而`first`是针对所有字段

**Rules**

最重要的还是Rules。定义rule可以有三种形式，但是rules字段名一定要和数据源source的字段名一致。
- 函数的方式：`{ name(rule, value, callback, source, options) {} }`
- 对象的方式： `{ name: { type: 'string', required: true } }`
- 数组的方式： `{ name: [{ type: 'string' }, { required: true }] }`

上面可以看到，字段名name可以有3种方式定义规则，在使用Element的时候还是推荐对象和数组的方式，这个两个比较简单，函数看情况使用。

除了`type`和`required`还有哪些用法可以用以及有什么用？

#### Rules的默认规则

- `type`: 要被验证的数据的类型如`url`和`number`等
- `required`： 是否必填
- `pattern`：使用正则来验证
- `min`: 数据的长度的最小值 (数据类型必须是`string`或`array`)
- `max`: 数据的长度的最大值 (数据类型必须是`string`或`array`)
- `len`: 数据的长度必须等于这个值 (数据类型必须是`string`或`array`)
- `enum`: 数据的值必须等于这个枚举数组某个元素 `{ enum: [1, 2, 3] }`
- `transform`: 一个钩子函数，在开始验证之前可以对数据先处理后验证，如吧`number`转为`string`后再验证
- `message`: 报错的提示信息可以是字符串也可以是jsx标签`<span>Name is required</span>`
- `validator`: 自定义验证函数以及报错信息 `validator(rule, value, callback)`
- 还有一个Deep Rules是处理`object`或者`array`类型的，使用了`fields` 或`defaultField`
- `fields`：Deep Rules的时候使用，定义下一层的字段名以及规则
- `defaultField`: Deep Rules的时候使用，所有下一层的字段都会采用该规则，可以被`fields`替换

#### 默认的Type
- `string`：必须是String类型，规则不设置type默认是这个
- `number`：必须是Number类型，如果后台返回的数据是字符串，可以用`transform`转为Number类型，字符串类型的数字（'12'）不会通过，要注意
- `boolean`: 必须是Boolean类型
- `method`: 必须是Function
- `regexp`：必须是正则RegExp
- `integer`：是Number类型的正整数
- `float`: 是Number类型的浮点数
- `array`: 是Array.isArray通过的数组
- `object`: Array.isArray不通过的Object类型
- `enum`: 要先定义enum，然后值必须是enum某个值
- `date`: 必须是Date对象的实例
- `url`: String类型且符合链接格式
- `hex`
- `email`: String类型，且符合邮箱格式

#### Deep Rules使用demo
```js
cosnt urls = ['http://www.baidu.com', 'http://www.baidu.com']
// 一个urls的数组，
const rules = {
  urls: {
    type: 'array',
    required: true,
    defaultField: { type: 'url' }
  }
}
```

```js
const ids = {
  name: 'LanTuoXie',
  age: 12,
  spc: '帅'
}

const rules = {
  ids: {
    type: 'object',
    required: true,
    fields: {
      name: { type: 'string', required: true },
      age: { type: 'number', required: true, tranform: Number },
      spc: { type: 'string', required: true }
    }
  }
}

```

#### 自定义验证validator
`validator(rule, value, callback)`
- `rule`: 记录了验证字段的字段名以及规则的信息
- `value`: 要验证的值
- `callback`: 如果`callback()`代表验证通过，如果`callback(new Error('错误要提示的信息'))`代表验证不通过

```js
// 验证是[min, max]范围内的正整数
const betweenInt = (min, max) => (rule, v, cb) => {
  const isBetween = v >= min && v <= max
  const isInt = /^[0-9]+$/.test(v)
  if (isBetween && isInt) return cb()

  return cb(new Error(`要求是在${min}到${max}的正整数 [${min}, ${max}]`))
}

const rules = {
  num: { validator: betweenInt(1, 5), required: true }
}
```
