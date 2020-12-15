# electron 常见功能

一般而言桌面端都拥有那些基本的功能？

## 操作系统

- `windows`
- `linux`
- `maxOS`

## `Window` 和 `Document`

`Window` 和 `Document` 主要存在于 `ipcRender` 进程中。

在 `Electron` 中，不要使用 `Document.addEventListener` 而是使用 `window.addEventListener`。

## `Main进程` 和 `Render进程` 通信

- 主进程和渲染进程主要通过 `channel` 相互通信。
- 进程回复主要通过 `event.reply()`  `event.returnValue` 的方式回复，又分异步和同步。
- 发送消息：`ipcRender.send()`  `ipcRender.sendSync()`

```js

// 在主进程中 replay() returnValue
const { ipcMain } = require('electron')
ipcMain.on('asynchronous-message', (event, arg) => {
  console.log(arg) // prints "ping"
  event.reply('asynchronous-reply', 'pong') // 异步回复
})

ipcMain.on('synchronous-message', (event, arg) => {
  console.log(arg) // prints "ping"
  event.returnValue = 'pong' // 同步
})

//在渲染器进程 (网页) sendSync() send()
const { ipcRenderer } = require('electron')
console.log(ipcRenderer.sendSync('synchronous-message', 'ping')) // prints "pong" 异步发送数据

ipcRenderer.on('asynchronous-reply', (event, arg) => {
  console.log(arg) // prints "pong"
})
ipcRenderer.send('asynchronous-message', 'ping') // 同步发送数据

```

## 于HTML区别的CSS

```css
/*是否可拖拽*/
.draggerable{
    -webkit-app-region: no-drag; // drag 可拖拽 no-drag 不可以拖拽
}

/*文本是否可选中*/
.user-select{
    -webkit-user-select: none; // 文本不可选中
}
```

## 系统通知 `Notification`

可以在主进程和渲染进程中添加。

```js
const myNotification = new Notification('Title', {
  body: 'Notification from the Renderer process'
})

myNotification.onclick = () => {
  console.log('Notification clicked')
}
```

## [无边框窗口](https://www.electronjs.org/docs/api/frameless-window)

无边框窗口是不带外壳（包括窗口边框、工具栏等），只含有网页内容的窗口

这是最简单，但是不是很推荐的方式。因为这个会让全局设置的快捷键失效。
```js
const { BrowserWindow } = require('electron')
const win = new BrowserWindow({ width: 800, height: 600, frame: false })
win.show()
```

你可能希望隐藏标题栏并将内容区域全屏的同时，依旧保留窗口控制按钮("红绿灯")来进行标准窗口操作。 你可以通过指定 `titleBarStyle` 选项来完成此操作

```js
const { BrowserWindow } = require('electron')
const win = new BrowserWindow({ titleBarStyle: 'hidden' })
win.show()
```

`hiddenInset` 返回一个另一种隐藏了标题栏的窗口，其中控制按钮到窗口边框的距离更大。

```js
const { BrowserWindow } = require('electron')
const win = new BrowserWindow({ titleBarStyle: 'hiddenInset' })
win.show()
```

`customButtonsOnHover`使用自定义的关闭、缩小和全屏按钮，这些按钮会在划过窗口的左上角时显示。

```js
const { BrowserWindow } = require('electron')
const win = new BrowserWindow({ titleBarStyle: 'customButtonsOnHover', frame: false })
win.show()
```

`透明窗口:` `transparent`

```js
const { BrowserWindow } = require('electron')
const win = new BrowserWindow({ transparent: true, frame: false })
win.show()
```

`点击穿透窗口：` 要创建一个点击穿透窗口，也就是使窗口忽略所有鼠标事件
```js
const { BrowserWindow } = require('electron')
const win = new BrowserWindow()
win.setIgnoreMouseEvents(true)
```

`可拖拽区：` `CSS -webkit-app-region: drag`

```html
<body style="-webkit-app-region: drag"></body>
```

```css
button {
  -webkit-app-region: no-drag;
}
```

`文本选择：`

```css
.titlebar {
  -webkit-user-select: none; // 文本不可选中
  -webkit-app-region: drag;
}
```

`右键菜单`：在某些平台上，可拖拽区域不被视为窗口的实际内容，而是作为窗口边框处理，因此在右键单击时会弹出系统菜单。 要使上下文菜单在所有平台上都正确运行, 您永远也不要在可拖拽区域上使用自定义上下文菜单。

## [主题色切换](https://www.electronjs.org/docs/tutorial/dark-mode)

主要切换APP背景颜色和字体颜色。

## remote

