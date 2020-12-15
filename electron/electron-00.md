# Electron 基本概念

## Windows Sandbox

`Windows Sandbox` 属于一个沙盘系统，许多人一提到沙盘，就会想到打仗时将领们商议战术时面前的沙盘，只是此沙盘并非彼沙盘，这里讲的沙盘，主要是可以帮助用户建立一个隔离的环境，以便用户在里边的所有操作不会影响到现有的操作系统。

`Chromium` 主要的安全特征之一便是所有的 `blink` 渲染或者 ``JavaScript` 代码都在sandbox内运行。 该sandbox使用OS特定特征来保障运行在渲染器内的进程不会损害系统。

Electron的一个主要特性就是能在`渲染进程`中运行`Node.js`（使用web技术能让我们更加便捷的构建一个桌面应用），但是在渲染进程中沙箱是不可用的。 这是因为大多数Node.js 的API都需要系统权限。 比如 ，没有文件系统权限的情况下require()是不可用的，而该文件系统权限在沙箱环境下是不可用的。

通常，对于桌面应用来说这些都不是问题，因为应用的代码都是可信的；但是显示一些不是那么受信任的网站会使得 `Electron` 相比 `Chromium` 而言安全性下降。 因为应用程序需要更多的安全性，sandbox 标记将使electron产生一个与沙箱兼容的经典chromium渲染器。

一个沙箱环境下的渲染器没有 `node.js` 运行环境，并且不会将Node.js 的 JavaScript APIs 暴露给客户端代码。 唯一的例外是预加载脚本, 它可以访问 `electron渲染器`  API 的一个子集(subset)。

## Spectron Devtron

这两者作为主程序的辅助功能。

- `Spectron` 测试工具
- `Devtron` debug工具

## [主进程 和 渲染进程](https://www.electronjs.org/docs/tutorial/application-architecture)

`Electron` 运行 `package.json` 的 `main` 脚本的进程被称为主进程。 在主进程中运行的脚本通过创建web页面来展示用户界面。 一个 `Electron` 应用总是有且只有一个主进程。

由于 `Electron` 使用了 `Chromium` 来展示 `web` 页面，所以 `Chromium` 的多进程架构也被使用到。 每个 `Electron` 中的 `web` 页面运行在它的叫渲染进程的进程中。

在普通的浏览器中，web页面通常在沙盒环境中运行，并且无法访问操作系统的原生资源。 然而 `Electron` 的用户在 `Node.js` 的 API 支持下可以在页面中和操作系统进行一些底层交互。

**区分主进程和渲染进程:**

主进程使用 `BrowserWindow` 实例创建页面。 每个 `BrowserWindow` 实例都在自己的渲染进程里运行页面。 当一个 `BrowserWindow` 实例被销毁后，相应的渲染进程也会被终止。

主进程管理所有的web页面和它们对应的渲染进程。 每个渲染进程都是独立的，它只关心它所运行的 `web` 页面。

在页面中调用与 `GUI` 相关的原生 `API` 是不被允许的，因为在 `web` 页面里操作原生的 `GUI` 资源是非常危险的，而且容易造成资源泄露。 如果你想在 `web` 页面里使用 `GUI` 操作，其对应的渲染进程必须与主进程进行通讯，请求主进程进行相关的 `GUI` 操作。

**主进程和渲染进程间通信**

`ipcRender` 和 `ipcMain` 模块发送信息。 使用 `remote` 模块进行RPC方式通信(Remote Procedure Call, 远程过程调用)

**主进程和渲染进程可以使用的API或者那些API不可使用**

所有 `Electron` 的API都被指派给一种进程类型。 许多API只能被用于主进程或渲染进程中，但其中一些API可以同时在上述两种进程中使用。 每一个API的文档都将声明你可以在哪种进程中使用该API。

**可以使用 `nodejs` 哪些API**

- 所有在Node.js可以使用的API，在Electron中同样可以使用。
- 你可以在你的应用程序中使用Node.js的模块。
- 原生Node.js模块 (即指，需要编译源码过后才能被使用的模块) 需要在编译后才能和Electron一起使用
- [加载原生node.js](https://www.electronjs.org/docs/tutorial/using-native-node-modules)

## 部署和打包

需要用到的库:

- `electron-forge`
- `electron-builder`
- `electron-packager`

这些工具将覆盖发布一个Electron应用所需采取的所有步骤，例如，打包应用程序，重组可执行程序，设置图标和可配置的创建安装程序。

打包流程：

为缓解 `Windows` 下路径名过长的 问题， 略微加快一下 require的速度以及隐藏你的源代码，你可以选择把你的应用打包成 `asar`档案文件，这只需要对你的源代码做一些很小的改动。

