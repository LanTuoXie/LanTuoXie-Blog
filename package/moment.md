# [moment.js](http://momentjs.cn/)

> JavaScript 日期处理类库

## 安装 moment.js

```bash
npm install moment --save   # npm
yarn add moment             # Yarn
Install-Package Moment.js   # NuGet
spm install moment --save   # spm
meteor add momentjs:moment  # meteor
bower install moment --save # bower (废弃)
```

## 使用简明指导

- 选环境安装
- 独立一个文件或者引入 `moment.js` 后，重新设置语言环境
- 封装一些常用的函数（结合使用的 UI 框架，如 Element UI 、 Ant UI），用于一些时间组件中。
- `moment` 是一个对象。类似 `jQuery` 的 `$`。执行完后返回一个 `moment object`。其实用法和 `jQuery` 差不多。
- 使用`new Date()`有`IOS`兼容问题，`moment`内部已处理(`new Date('YYYY-MM-DD HH:mm:ss')`，IOS不兼容`YYYY-MM-DD`要将其转化未`YYYY/MM/DD`)

## moment Doc 简明简介

简明介绍`moment`由那部分组成。

- 解析： 将符合`moment语法的时间格式`解析且存于`moment object`内部的值。类似`$('.className .className')`。都有自己的一套解析语法，从官方文档中选择你需要的时间格式即可。常见的有`moment('YYYY-MM-DD HH:mm:ss')`、`moment({ hour:15, minute:10 })`、`moment(1318781876406)`等。
- 取值/赋值：`moment()`后返回的`moment object`内部存储了日期和时间的值，类似`jQuery`其有自带的方法，可以操作封装在其内部的值。
- 操作：是`moment object`暴露给用户的一些方法函数。可以更加方便的操作和改变`moment object`内部的值。
- 显示：将用户操作后的`moment object`转化为各种格式的日期时间。如`valueOf 时间戳`、`format('YYYY-MM-DD HH:mm:ss') 日期时间字符串`等。
- 查询：封装了一些用于判断日期时间的`断言函数`。用于判断`某个日期`是否处于`某个日期之前或者之间`等，很有用。如`结束时间`是否大于`开始时间`。
- 国际化：用于配置不同语言的基本配置。比如表示星期的值，默认是从`周日开始的`，且`周日的值是 0`，如果想一周的开始时间从`周一`开始，就需要配置这里了。
- 时长：`moment object`暴露一些方法，让操作者更易获取其内部的`年、月、日`等表示一段时间的值。

## 一些有用的代码片段

