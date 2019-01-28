# Canvas

# canvas 画布

canvas基本功能的总结和一些问题解决笔记

## canvas基本知识点

- canvas的默认宽高是300x150px，可以通过设置width和height属性来自定义宽高
- 替换内容：如果浏览器不兼容会渲染`<canvas>如果浏览器不兼容，将看到这段话</canvas>`被canvas标签包含的内容
- 一定要`</canvas>`结尾标签，不可以和图片`<img />`那样`<canvas />`，也即必须`<canvas></canvas>`

## 渲染上下文render context

- 渲染上下文分2D和3D
- `canvas.getContext('2d')`用于获取2d的渲染上下文，就是获取一个2d画板的实例
- 只有先获取这个上下文才可以后续的操作，这个操作是必须的，就如要操作dom，要先获取dom`document.getElementById('id')`
- 这个上下文中包含了后面一系列操作的方法和属性

## 画笔的笔头

- 画笔的`笔头`是各种形状的图像，如`矩形、三角形、直线、圆弧、曲线`
- `绘画`是由`笔头`多个点连接而成的某个形状，要绘画，首先要选择要绘画的笔头
- 举一个现实中的例子：装修工人用粉刷刷墙，那么这个`笔头`是什么呢？`笔头`可以是`直线、或者长方矩形`

## 坐标

- canvas的坐标(x, y)位于左上角
- canvas的原点坐标(0, 0)
- 所有绘画到画布上的形状都相对于canvas的原点(0, 0)

## 矩形笔头

- `fillRect(x, y, width, height)`：绘制一个填充的矩形（实心矩形笔头，现实中，一个矩形的锤子）
- `strokeRect(x, y, width, height)`：描边一个矩形的边框（空心矩形笔头，现实中，做爱心饼干的那种空心工具）
- `clearRect(x, y, width, height)`：清除指定矩形区域，让清除部分完全透明（矩形的橡皮檫）

## 路径函数（路径由一连串的矩形点组成）

- `beginPath()`：新建一条路径，并配合`closePath()`来生成一条指定的路径
- `closePath()`：闭合路径
- `stroke()`：路径的轮廓
- `fill()`：填充路径
- 路径列表：在未`fill()`之前，路径列表用于收集要绘制路径的点，以及绘制路径的方法

注意点：

- 本质上，路径是由很多子路径构成，这些子路径都是在一个列表中，所有的子路径（线、弧形、等等）构成图形。
- 闭合路径closePath(),不是必需的。
- 当调用`fill()`的时候，所有没有闭合的形状都会自动闭合，但是调用`stroke()`时不会自动闭合

```js
  ctx.beginPath();
  ctx.moveTo(75, 50);
  ctx.lineTo(100, 75);
  ctx.lineTo(100, 25);
  ctx.fill();
```

- `moveTo(x, y)`：将笔触移动到指定的坐标x以及y上
- `lineTo(x, y)`：画线

## 圆弧路径 circle arc

基本弧线：

- `arc(x, y, radius, startAngle, endAngle, anticlockwise)`：绘画一个弧度
- 要画一个弧度，最基本的圆心点(x, y)以及弧度radius
- 然后就是弧长，弧长由弧度决定，也即`endAngle - startAngle`来决定，圆的弧度是`2PI`，半圆是`PI`
- 最后一个是绘画的方向，`anticlockwise`是顺时针还是逆时针，值是bool值
- `弧度=(Math.PI/180)*角度`
- `arcTo(x1, y1, x2, y2, radius)`：根据给定的控制点和半径画一段圆弧，再以直线连接两个控制点。

## 贝塞尔曲线

- `quadraticCurveTo(cp1x, cp1y, x, y)`：绘制二次贝塞尔曲线，cp1x,cp1y为一个控制点，x,y为结束点。
- `bezierCurveTo(cp1x, cp1y, cp2x, cp2y, x, y)`：绘制三次贝塞尔曲线，cp1x,cp1y为控制点一，cp2x,cp2y为控制点二，x,y为结束点。

## 常用的矩形路径

- `rect(x, y, width, height)`：绘制一个左上角坐标为（x,y），宽高为width以及height的矩形。

## Path2D 对象 (用来缓存或记录绘画命令，这样你将能快速地回顾路径)

- `new Path2D()`：创建一个path2D实例
- `Path2D.addPath(path [, transform])​`：添加了一条路径到当前路径（可能添加了一个变换矩阵）
- path2D实例可以作为`fill(path)、stroke(path)`来生成路径
- 也就是说path2D就是一个路径列表，用于保存要绘画的固定的路径，可以多次绘画

## 色彩 （填充和描边）

- `fillStyle = color`：填充的颜色
- `strokeStyle = color`：描边的颜色
- color是css规范的颜色值
- 如果你要给每个图形上不同的颜色，你需要重新设置 fillStyle 或 strokeStyle 的值。

## 透明度

- `globalAlpha = transparencyValue`：全局的透明度
- 当然也可以使用rgba

## 线的形状（路径）

实线：

- `lineWidth = value`：线的宽度
- `lineCap = type`：线的末端
- `lineJoin = type`：两条线的节点
- `miterLimit = value`：限制当两条线相交时交接处最大长度；所谓交接处长度（斜接长度）是指线条交接处内角顶点到外角顶点的长度。

虚线：

- `getLineDash()`：返回一个包含当前虚线样式，长度为非负偶数的数组。
- `setLineDash(segments)`：设置当前虚线样式
- `lineDashOffset = value`：设置虚线样式的起始偏移量

公用：

- `lineCap`的可能值：butt（默认）、round（圆角） 、square
- `lineJoin`的可能值：miter（默认：尖角）、bevel（磨平）、round（磨圆）

## 渐变

- `createLinearGradient(x1, y1, x2, y2)`：线性渐变，直线渐变，由这两点的直线与水平线的夹角形成渐变方向，线的长度绝点渐变的强大
- `createRadialGradient(x1, y1, r1, x2, y2, r2)`：径向渐变

## 图片 pattern（图片背景）

- `createPattern(image, type)`：创建一个图片
- 其中image要求四Image对象的实例
- type：repeat、repeat-x、repeat-y、no-repeat
- 与 drawImage 有点不同，你需要确认 image 对象已经装载完毕，否则图案可能效果不对的。

## 阴影

- `shadowOffsetX = float`：x轴偏移
- `shadowOffsetY = float`：y轴偏移
- `shadowBlur = float`：模糊
- `shadowColor = color`：阴影颜色
- 没有扩散，有模糊

## 文本

绘画文本：

- `fillText(text, x, y [, maxWidth])`：在指定的(x,y)位置填充指定的文本，绘制的最大宽度是可选的
- `strokeText(text, x, y [, maxWidth])`：在指定的(x,y)位置绘制文本边框，绘制的最大宽度是可选的

文本的样式：

- `font = value`：默认的字体是 10px sans-serif。
- `textAlign = value`：可选的值包括：start, end, left, right or center. 默认值是 start。
- `textBaseline = value`：可选的值包括：top, hanging, middle, alphabetic, ideographic, bottom。默认值是 alphabetic。
- `direction = value`：可能的值包括：ltr, rtl, inherit。默认值是 inherit。

更多文本的细节

- `measureText()`：将返回一个 TextMetrics对象的宽度、所在像素，这些体现文本特性的属性。

## 图片

- `drawImage(image, x, y)`：绘制图片
- `drawImage(image, x, y, width, height)`：后面两个用于缩放
- `drawImage(image, sx, sy, sWidth, sHeight, dx, dy, dWidth, dHeight)`：后面的参数用于切片，裁剪

## 快照（快照栈）

- `save()、restore()`：save 和 restore 方法是用来保存和恢复 canvas 状态的
- Canvas 的状态就是当前画面应用的所有样式和变形的一个快照。
- Canvas状态存储在栈中，每当save()方法被调用后，当前的状态就被推送到栈中保存。
- 每一次调用 restore 方法，上一个保存的状态就从栈中弹出，所有设定都恢复。

状态保存的是什么？

- 当前应用的变形，也就是原点坐标的位置不改变，以及旋转、缩放等
- 一些样式，配色方案：`strokeStyle、fillStyle`
- 阴影：`shadowOffsetX、shadowOffsetY、shadowBlur、shadowColor`
- 透明度：`globalAlpha `
- 线的样式：`lineWidth、lineCap、lineJoin、miterLimit`
- 图片合成的模式：`globalCompositeOperation `
- 裁剪路径：`clip()`

## transform（操作原点）

- `translate(x, y)`：移动 canvas 和它的原点到一个不同的位置。
- `rotate(angle)`：旋转
- `scale(x, y)`：缩放
- `transform(m11, m12, m21, m22, dx, dy)`：
- `resetTransform()`：重置原点位于左上方
- 由于这些操作都是相对于原点坐标的，可以通过translate来一定坐标原点，然后再绘画

## 合成和裁剪

- `globalCompositeOperation = type`：这个属性设定了在画新图形时采用的遮盖策略，其值是一个标识12种遮盖方式的字符串。
- `clip()`：将当前正在构建的路径转换为当前的裁剪路径。

## 动画

基本操作：

- 清空canvas
- 保存canvas状态
- 绘制动画图形
- 恢复canvas状态

定时器:

- `setInterval(function, delay)`
- `setTimeout(function, delay)`
- `requestAnimationFrame(callback)`

缓动（添加速率）：

- 缓动函数，也就是物体运动的加速度，就是`s和t的比例 (s2 - s1)/ (t2 - t1)`
- `s2`是终点,`s1`是起点，`t1`是开始时间，`t2`是结束时间

边界

- 任何运动都要考虑边界条件
- 一般分析的方式是几何分析

长尾效果

- 使用`clearRect`函数清除前一帧动画。一个半透明的`fillRect`函数取代之，就可轻松制作长尾效果。

## 像素操作

## 保存图片

- `canvas.toDataURL('image/png')`：保存为base64
- `canvas.toDataURL('image/jpeg', quality)`：质量是0-1
- `canvas.toBlob(callback, type, encoderOptions)`：创建了一个在画布中的代表图片的Blob对像

## 优化

- 在离屏canvas上预渲染相似的图形或重复的对象：创建多个canvas，但是那个canvas是隐藏的，用于绘画一些不变的图像或者重复的，当需要的时候从中取出，也就是图形仓库
- 避免浮点数的坐标点，用整数取而代之：坐标尽量以整数表示，避免使用浮点数，js的浮点数运算有bug
- 不要在用drawImage时缩放图像
- 使用多层画布去画一个复杂的场景：一些固定不变的画面尽量用别的画布绘画或者用dom配合css去绘制，因为每次绘画都要清空绘画
- 用CSS设置大的背景图：大图很费性能，最好用dom去处理，加上它由不变的
- 用CSS transforms特性缩放画布：transform可以启动GPU
- `getContext('2d', { alpha: false })`用于关闭透明度
- 将绘画的操作步骤封装成函数
