# Css规范

- `OOCSS`
- `SMACSS`
- `BEM`

## OOCSS(Object Oriented CSS)面向对象的css

主要分成四个部分

- `Template` ：模板
- `Grids` ：栅格布局
- `Module` ：组件
- `Content` ：内容

**Template模板:**

主要负责`HTML的结构`，让`HTML`更有层次感。<br>
可以按`功能模块`来构建HTML的结构<br>
常见的如：

人体架构

- `.header`
- `.body`
- `.footer`

垂直方向

- `.top`
- `.middle`
- `.bottom`

水平方向

- `.left`
- `.main`
- `.right`

事物的结构构造（房子）由外向内

- `.building`
  - `.level`
    - `.room`
      - `.door`
      - `.window`
      - `.wall`
      - `.inner`

结构整么搭，要根据业务来构建，以上只供参考

**Grids栅格布局:**

主要是布局方面的类如何定义，可能采用的布局方式不同而不同。
参见的css布局：

- `浮动布局`
- `弹性盒子布局`
- `Grid布局`

`Grid布局`由于兼容性问题，现在大多采用`浮动布局`和`弹性盒子`(IE10+ , 移动端)<br>

浮动布局

- `.line`
  - `.unit`
  - `.lastUnit`
  - `.size1of2`
  - `.size2of3`

```scss
//结构 也叫容器
.line {
  position:relative;
}

.line:after {
  content: ' ';
  display: 'block';
  height: 0;
  clear:both;
  visibility: hidden;
}

//浮动
.unit {
  float: left;
}

//添加到最后一个 unit后
.lastUnit {
  display: table-cell;
  float: none;
  width: auto;
}

//unit 的比例 1/2 
.size1of2 {
  width: 50%;
}

//unit 的比例2/3
.size2of3 {
  width: 66.6%;
}
```

浮动布局清浮动还可以参考`bootstrap3` 更优雅

```scss
.line:before,
.line:after {
  content: '';
  display: table;
}

.line:after {
  clear: both;
}
```

弹性盒子参考`bootstrap4`

- `.container`
  - `.row`
    - `.col`

**Module组件:**

各个组件特有的结构和样式。<br>
以下类：

- `btn`
- `btn btn-error`
- `modal md-title`

这里只要使用了继承的机制
`btn-error`继承了`btn`的基本样式，然后扩展了自己的样式`error` <br>
`md-title`继承了`modal`的基本样式，然后添加自己专属样式
在`sass`中，可以通过占位符`%`和搭配`@extend`实现

**Content内容:**

定义html标签，适合自己应用的基本样式，不用类，直接定义标签的样式。

## [SMACSS可扩展模块化架构的CSS](https://smacss.com/)

[smacss免费在线书籍(英)](https://smacss.com/book/)

SMACSS的5大核心分类

- `Base`
- `Layout`
- `Module`
- `State`
- `Theme`

**Base:**

负责定义页面的默认样式，由于不同的浏览器都有自己的默认样式，要做到在不同浏览器上效果一致，需要开发者重写这些样式。<br>
常见的base配置有[Reset.css](https://meyerweb.com/eric/tools/css/reset/) 和 [normalize.css](http://necolas.github.io/normalize.css/)<br>
由于中文和英文排版和显示有区别，推荐参考妹子ui的base。<br>
base文件的规范：该文件只使用`标签选择器`、`标签选择器配伪类`、`*`，`!important`不应该出现在该文件中

```scss
*{
    margin:0;
    padding:0;
    border:0;
}

*,
*:before,
*:after {
  box-sizing: border-box;
  -webkit-tap-highlight-color: transparent;
}

html,
body {
  min-height: 100%;
  background-color: #fff;
}

a:active,
a:hover {
  outline: 0;
}

```

**Layout:**

主要负责页面的布局，在布局的时候，我们首先对页面进行结构布局，如圣杯布局

- `#header`
- `#article`
  - `#sidebar`
  - `#main`
- `#footer`

对于`整体的结构布局`，由于只出现一次，且只能有一个，这里要使用`id选择器`。

```css
#header, #article, #footer {
    width: 960px;
    margin: auto;
}

#article {
    border: solid #CCC;
    border-width: 1px 0 0;
}
```

对于`行列的布局`，也即`局部的布局`，这些可以多次重复使用的，就不能使用`id选择器`了。<br>
通过采用前缀的方式区分，`l-`，只要是和布局相关的都要加`l-`前缀，且只能是和布局相关的才能使用这个前缀。<br>

常见的有栅格布局，按行和列布局等，可以参考上面OOCSS的Grids栅格布局，只需添加前缀`.l-line`和`.l-unit`。<br>
还有，由于列表`ul 或 ol`是我们页面中最常用结构，很有必要给这个结构设计一个布局方式。

```scss
.l-list{
    margin: 0;
    padding: 0;
    list-style-type: none;
}

.l-list-item{
    float: left;
    height: 100px;
    margin-left: 10px;
}
```

`.l-list(ul/ol)`、`.l-list-item(li)`比如上面这个浮动布局，这里没有清浮动，还可以用弹性盒子等布局方式。<br>

为避免布局混乱，加强结构的层次，可以采用`子选择器: >`，来强耦合布局和HTML结构

```scss
.l-list{ 
    margin: 0;
    padding: 0;
    list-style-type: none;
}

.l-list > .l-list-item{
    float: left;
    height: 100px;
    margin-left: 10px;
}
```

上面的结构强绑定对应的html结构才生效

```html
<ul class='l-list'>
  <li class='l-list-item'></li>
  <li class='item'>
    <ul class='list'>
      <li class='l-list-item'></li>
    </ul>
  </li>
</ul>
```

如上面这个，最外层`<ul class='l-list'>`的 `<li>`只有类为`.l-list-item`才生效<br>
而内部的`<ul class='list'>`的`<li class='l-list-item'>`就不会生效，`l-list-item`的父类一定要为`.l-list`才生效<br>

但是`>`也有其缺点

- `IE6不兼容`
- `会增加css文件的体积和复杂性`

优点

- `可以用在任何地方，只要html符合格式`
- `性能比使用后代选择器或元素选择器更好(计算CSS值的时候)`

**Module:**

避免使用`id选择器`和`标签选择器`，应该只使用`类选择器`。<br>
module也即组件，组件应该独立，有自己的命名空间。这个和OOCSS的module一样。<br>
如一个模态窗：

```scss
.modal{}

.mod-header{}

.mod-body{}

.mod-footer{}
```

```html
<div class='modal'>
  <div class='mod-header'></div>
  <div class='mod-body'></div>
  <div class='mod-footer'></div>
</div>
```

**State:**

个人觉得SMACSS最有意思的地方有，一是通过`特殊的前缀`指定命名空间和划分功能模块，再就是这个State。<br>
表示某个指定状态下的样式，如

- `.is-collapsed`
- `.is-clicked`
- `.is-error`
- `.is-success`
- `.is-hidden`

在模块中也可以定制特殊的状态。

```css
.tab {
    background-color: purple;
    color: white;
}

.is-tab-active {
    background-color: white;
    color: black;
}
```

这样就可以参考事物运动的过程，以`状态切换点`分割运动过程，不但可以减少思维上的复杂性，还利于维护。<br>
特别是用户频繁操作的界面。

涉及到状态改变有四种方式

- `类名 class name`
- `属性选择器 加 data-`
- `伪类:hover :focus`
- `媒体查询@media`

通过改变类名的方式改变状态<br>
通过定义伪类:hover :focus等有状态含义的伪类<br>
通过定义媒体查询，根据屏幕大小变化来改变状态<br>
<br>
其中，可以通过改变类名，删除、添加、替换的方式切换状态。<br>
类中可以包含动画，伪类:after :before，还可以通过添加data-来区分不同的状态。

**Theme:**

重写各个组件的基本样式，以达到切换主题的目的

```scss
// in module-name.css
.mod {
    border: 1px solid;
}

// in theme.css
.mod {
    border-color: blue;
}
```

不过本人更喜欢`bootstrap`的`variables.scss`的方式

## [BEM](http://getbem.com/)

Block 、 Element 、 Modifier<br>
块  >> 元素  >> 修饰符<br>

```scss
.form { }

.form--theme-xmas { }
.form--simple { }

.form__input { }
.form__submit { }

.form__submit--disabled { }
```

```html
<form class="form form--theme-xmas form--simple">
  <input class="form__input" type="text" />
  <input
    class="form__submit form__submit--disabled"
    type="submit" />
</form>
```

Block: `.form`<br>
Element: `.form__input` `.form__submit`<br>
Modifier: `.form--theme-xmas` `.form--simple`

一个Block由多个Element组成<br>
Element由Block名充当前缀，且用`__`分隔符来区分Element<br>
Modifier类似SMACSS的State，由Block名充电前缀，且用`--`分隔符来区分Modifier<br>
<br>
上面中有个`.form__submit--disabled`<br>
代表form块(Block)的submit元素(Element)的disabled修饰符(Modifier)

## 总结

- `不管是SMACSS 还是 BEM 都采纳了OOCSS`
- `OOCSS是所有的基础`
- `SMACSS的分类方式，且提出的State很赞`
- `BEM 用于写组件是个很不错的方式，不过类名太长了`
- `结合三者，OOCSS为基础，按SMACSS划分项目结构，BEM写组件`

```scss
.modal{}

.modal > .mod_header{}
.modal > .mod_body{}
.modal > .mod_footer{}

.modal > .mod_header-active{}

.mod_header > .header-hidden{}

.mod_body > .body_title{}
.mod_body > .body_title > .title-big{}
```

通过`子选择器`保存之前的命名空间，然后再开一个前缀，这样就可以避免类名过长<br>
但是使用`>`会增加css文件的体积。
