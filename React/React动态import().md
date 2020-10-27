# React动态import()

`react-router@v4`代码分离，推荐的import()。这里分享webpack配置和使用方法。

首先安装两个必须的包

```bash
cnpm i react-loadable babel-plugin-syntax-dynamic-import -D
```

react-loadable是官方推荐的动态加载组件，babel-plugin-syntax-dynamic-import是babel支持webpack的import()插件。
配置方法：在`.babelrc`

```bash
{
  "presets": [
    "react"
  ],
  "plugins": [
    "syntax-dynamic-import"
  ]
}
```

上面`babel-plugin`前缀是可以省略的。

上面配好后，如果你配了eslint还是会报错的，如果eslint配置不对。报`import() undefined`
要保证eslint不报import()错误需要
`cnpm i babel-eslint -D`

然后在`.eslintrc`加上配置

```js
module.exports = {
  //...若干配置
  
  parser: "babel-eslint"
}
```

使用babel-eslint解析才可以识别import()

然后就是使用了^_^。

这里我把官方的demo封装成了一个文件`loadable.jsx`

```js
import React from 'react'
import Loadable from 'react-loadable'

const Loading = () => {
  return <div>loading...</div>
}

export default (Loader) => {
  const LoadableComponent = Loadable({
    loader: Loader,
    loading: Loading
  })

  return class LoadableHOC extends React.Component {
    render () {
      return <LoadableComponent></LoadableComponent>
    }
  }
}
```

随便写一个需要动态导入的组件`Import.jsx`

```js
import React from 'react'

class Import extends React.Component {
  render () {
    return <div>import...</div>
  }
}

export default Import

```

包含的动态导入的容器组件`Test.jsx`

```js
import React from 'react'
import loadable from '@/utils/loadable'

const Import = loadable(() => import('@/components/Import'))

class Test extends React.Component {
  render () {
    return (<div>
      <Import></Import>
    </div>)
  }
}

export default Test

```

然后在`main.jsx`

```js
import React from 'react'
import ReactDom from 'react-dom'
import Test from '@/components/Test'

ReactDom.render(<Test />, document.getElementById('app'))

```
