## Element表单验证（2）

上篇讲的是`async-validator`的基本要素，那么，如何使用到Element中以及怎样优雅地使用，就在本篇。

上篇讲到`async-validator`由3大部分组成
- `Options`
- `Validate`
- `Rules`

基本验证流程如下
- 先按照rule的规则，制定每个字段的规范，生成rules
- 根据rules生成验证器`const validator = new Validator(rules)`
- 验证器有验证函数`validator.validate(source, callback)`
- source中的字段对应规则中的字段，全都通过或出错后调用callback

上面中的`validator.validate`对应Element中的`this.$refs[refName].validate`，只不过被改装过的。而且在Element中定义`<el-form :model='form'>` 的`:model='form'`，那个`form`就是`source`。`source`的字段名，如`source.name`就是`form.name`，那么只要在`<el-form-item prop='name'>`设置和`source`一样的字段名`name`，就可以匹配`<el-form :rules='rules'>`中的`rules.name`。

很重要的一点，`rules.字段名`要和`source.字段名`要一样才会验证。
```html
<template>
  <el-form :model='form' ref='domForm' :rules='rules'>
    <el-form-item prop='name' lable='名字'>
      <el-input v-model='form.name'>
    </el-form-item>
  </el-form>
</template>
```
```js
export default {
  data() {
    this.rules = {
      name: { type: 'string', required: true, trigger: 'blur' }
    }

    return {
      form: {
        name: ''
      }
    }
  },
  methods: {
    submit() {
      this.$refs.domForm.validate(valid => {
        if (!valid) {
          // 验证不通过
        }

        // 验证通过后的处理...
      })
    }
  }
}
```
上面中`validate(callback)`函数已经添加到form元素DOM节点上的属性中。然后上面还有一个不好的一点。那就是规则的定义方式不够灵活。

比如我有两个字段`firstName`和`lastName`。`firstName`是必填的，而`lastName`是非必填的；且`firstName`长度要求大于1小于4而`lastName`要求大于1小于6。那么就要写两个不同的规则，现在只是2个字段而已，没什么，如果有很多个不同要求的字段，那要写很多个不同的规则，也要写很多个规则？岂不是很烦？能不能复用？

**Rules三种定义方式**
- 函数的方式：`{ name(rule, value, callback, source, options) {} }`
- 对象的方式： `{ name: { type: 'string', required: true } }`
- 数组的方式： `{ name: [{ type: 'string' }, { required: true }] }`

函数的方式很强大，如果需要用到`options`可以函数的方式，最常用的是对象和数组的方式。现在推荐几种复用的方法。

#### 简单的封装一些常用的规则
```js
// 返回一个规则数组，必填且字符串长度在2~10之间
export const name = (msg, min = 2, max = 10, required = true) => ([
  { required, message: msg, trigger: 'blur' },
  { min, max, message: `长度在${min}~${max}个字符`, trigger: 'blur' }
])

// 邮箱
export const email = (required = true) => ([
  { required, message: '请输入邮箱', trigger: 'blur' },
  { type: 'email', message: '邮箱格式不对', trigger: 'blur' }
])

/* 自定义验证规则 */

// 大于等于某个整数
const biggerAndNum = num => (rule, v, cb) => {
  const isInt = /^[0-9]+$/.test(v)
  if (!isInt) {
    return cb(new Error('要求为正整数'))
  }

  if (v < num) {
    return cb(new Error(`要求大于等于${num}`))
  }
  return cb()
}

export const biggerInt = (int, required = true) => ([
  { required, message: '必填', trigger: 'blur' },
  { validator: biggerAndNum(int), trigger: 'blur' }
])

```

#### 封装一个专门创建规则的类，链式调用
```js
export default class CreateRules {
  constructor() {
    super()
    this.rules = []
  }

  need(msg = '必填', trigger = 'blur') {
    this.rules.push({
      required: true,
      message: msg,
      trigger
    })
    return this
  }
  
  url(message = '不是合法的链接', trigger = 'blur') {
    this.rules.push({
      type: 'url',
      trigger,
      message
    })
    return this
  }

  get() {
    const res = this.rules
    this.rules = []
    return res
  }
}

const createRules = new CreateRules()

//规则
const needUrl = createRules.need().url().get()
const isUrl = createRules.url().get()

// imgUrl必填且是链接；href可选不填，如果填写必须是链接
const rules = {
  imgUrl: needUrl,
  href: isUrl
}

// deep rule 定义
// urls是数组，长度大于1
// urls的元素是链接
const urls = ['http://www.baidu.com', 'http://www.baidu.com']

const rules = {
  urls: {
    type: 'array',
    min: 1,
    defaultField: isUrl
  }
}
```
