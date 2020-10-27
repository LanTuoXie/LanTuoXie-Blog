# Sass 项目结构之7-1模式

`7-1模式`的结构：`7个文件夹`，`1个文件`。<br>
基本上，你需要将所有的部件放进7个不同的文件夹和一个位于根目录的文件（通常用`main.scss`或者`app.scss`）<br>
编译时会根据`main.scss`引用所有文件夹而形成一个CSS样式表

## 7个文件夹

- `abstracts/`
- `base/`
- `components/`
- `layout/`
- `pages/`
- `themes/`
- `vendors/`

## 1个文件

- `main.scss`

## Base文件夹

`base/`文件夹存放项目中的模板文件。定义一些HTML元素公认的标准样式。<br>
比如：[Reset.css](https://meyerweb.com/eric/tools/css/reset/) 和 [normalize.css](http://necolas.github.io/normalize.css/) 等

- `_base.scss`
- `_reset.scss`
- `_typography.scss`
- `_animations.scss`

## Layout文件夹

`layout/`文件夹存放构建网站或者应用很小使用到的布局部分。常见的如：

- `_grid.scss`
- `_header.scss`
- `_footer.scss`
- `_sidebar.scss`
- `_forms.scss`
- `_navigation.scss`

## Components文件夹

该文件夹包含各类具体模块，基本上是所有的独立模块，划分可以参考一些UI库的来划分，如:

- `_media.scss`
- `_carousel.scss`
- `_thumbnails.scss`
- `_alert.scss`
- `_badge.scss`
- `_button.scss`
- `_modal.scss`

等等，这个文件夹理应做到每个文件对应一个独立的组件，或者在该目录下再开个目录`button/`<br>
用`components/`目录下的`_button.scss`做链接，链接`button/`中所有文件<br>

>`_button.scss` 文件只负责`@import`，而实际功能在文件夹`button/`所有文件中<br>
>一般做链接的文件不该用`_`前缀，而是用`button.scss`来区分，实际功能才用`_`前缀

```css
/* button.scss */

@import "./button/_blue-button";
@import "./button/_red-button";
```

---

## Pages文件夹

存放页面文件，就是那个页面内具有独特的样式，如：

- `_home.scss`
- `_index.scss`

---

## Themes文件夹

主题文件夹，类似`Bootstrap`中的`_variables.scss`文件,存放着整个app的变量，组件的样式值基本引用于该文件。<br>
只要改变`_variables.scss`中的值，就可以切换一种主题。

- `_theme.scss`
- `_admin.scss`
- `_variables.scss`

---

## Abstracts文件夹

辅助工具文件夹，可以存放每一个全局变量、函数、混合宏以及占位符。如：<br>

- `_variables.scss`
- `_mixins.scss`
- `_functions.scss`
- `_placeholders.scss`

文件夹名字和这些文件只是参考，不喜欢也可以`utils`,如果`_mixins.scss`很大，<br>
可以在该文件夹下再创一个`mixins/`，`mixins.scss`充当链接`@import`

---

## Vendors文件夹

外来项目文件夹，存放一些外部库和框架`(Normalize,Bootstrap,jQueryUI等)`

- `_normalize.scss`
- `_bootstrap.scss`
- `_jquery-ui.scss`

---

## 入口文件

主文件，`main.scss`，除`@import`和注释外，该文件不应该包含任何其他代码<br>

为了保持可读性，主文件应遵循如下准则:

- 每个`@import`引用一个文件；
- 每个`@import`单独一行；
- 从相同文件夹中引入的文件之间不用空行；
- 从不同文件夹中引入的文件之间空行分隔；
- 忽略文件扩展名和下划线前缀

>下面以一个单页面`app.scss`文件为例<br>
>这个案例的模式我是参考bootstrap的框架的，非`7-1模式`，任何模式都只供参考<br>
>具体怎么用，请按具体业务应用使用

- `_global.scss`
- `_animations.scss`
- `app.scss`
- `mixins/`
- `functions/`
- `components/`
- `containers/`

```css
/*
 * 参考bootstrap架构
 * xxx : 表示同理
 * app.scss
 */

/* 导入变量 */
@import 'variables';

/* 导入mixins */
@import './mixins/clearfix';
@import './mixins/xxx';

/* 导入functions */
@import './functions/px2rem'
@import './functions/xxx';

/* 导入动画 */
@import 'animations';

/* 导入基本设置，排版、html标签等全局样式 */
@import 'global.scss';

/* 导入components */
@import './component/button';
@import './component/xxx';

/* 导入containers */
@import './containers/index';
@import './containers/xxx';
```

---

## 总结

`7-1模式`，只供参考，具体怎么搭，最好按项目来。<br>
如果项目比较小，用`7-1模式`就不必要了，参考`bootstrap`的架构就可以了<br>
如果项目很大，采用`7-1模式`是个不错的选择。<br>
最主要的是学习其中项目管理的思想。

---

## 参考资料链接

- [Sass Guidlines](https://sass-guidelin.es/#options-panel)
- [中文版(大漠.译)](https://github.com/HugoGiraudel/sass-guidelines/tree/master/pages/zh)
