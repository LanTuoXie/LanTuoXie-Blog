# React-Router笔记

## <Route>

`要把<Route>当成是一个高阶组件`

拥有的属性：

- `render: func`
- `children: func`
- `path: string`
- `exact: bool`
- `strict: bool`
- `location: bool`
- `sensitive: bool`

有3种方式渲染一些视图通过`<Route>`，但是这种方式只能选择一种，不能重复，会有优先级    
使用最多的是`component`的方式，也是以前我们的用法

> 三者不可以同时使用，优先级component > render > children

- `<Route component>`
- `<Route render>`
- `<Route children>`

有3个`props`会默认传给`<Route>`的组件中的`props`，也即3种渲染方式的`component、render、children`都能拿到

- `match`
- `location`
- `history`

`<Route path='/shop' component={ ShopIndex }>`：优选最高，当使用这个，优选使用这个方式渲染

```js
// 上面那段代码的意思相当于
{
 render() {
    if (currPath !== '/shop') {
    return null
    }
    return <ShopIndex />
 }  
}
```

- `<Route render>`： 就是自己定义渲染函数，也就开放`render`函数给开发者自己定义怎么渲染
- `<Route children>`：和其它两个不同的是，它可以渲染匹配的和不匹配的，而之前两个只渲染不匹配的，也就是说，就算路由不匹配你也可以渲染该组件，其实最重要的是该组件拿到了`Route`的3个属性
- `path`：如果一个`<Route>`不加`path`代表永远匹配
- `exact`：如果有一个路由`/one/two`，当当前路由是`/one`，在`exact`是`true`时，不匹配，当`false`时匹配；这里比较的是当前路由和`location.pathname`，当是有种情况是`currPathname + '/'`也是匹配的
- `strict`: 相对于上面，严格要求当前页面的`location.pathname === currPathname`
- `sensitive`: 路由区不区分大小写

## <Switch>

被`<Switch>`包含的`<Route>`只会渲染一个

```js
<Switch>
  <Route exact path="/" component={Home}/>
  <Route path="/about" component={About}/>
  <Route path="/:user" component={User}/>
  <Route component={NoMatch}/>
</Switch>
```

比如上面的结构，其实就如下面这段代码

```js
switch (currPath) {
  case '/': {
    return <Home />
  }
  
  case '/about': {
    return <About />
  }
  
  case '/:user': {
    return <User />
  }
  
  default: {
    return <NoMatch />
  }
}
```

注意是`return`，所以`<Switch>`后只有一个路由组件被渲染

## withRouter

这个高阶组件主要是给组件的`props`注入`match、location、history`，如果有哪些组件想拿到当前路由的信息可以使用这个

> 如果组件还使用了redux，要这样使用

```js
import { withRouter } from 'react-router'

withRouter(connect(...)(MyComponent))
```
