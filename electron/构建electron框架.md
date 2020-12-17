# 构建Electron框架

## main.js

加载基本静态文件 `index.html`：有服务器版本

```js
let mainWindow
const winURL = process.env.NODE_ENV === 'development'
  ? `http://localhost:9080`
  : `file://${__dirname}/index.html`
```

```js
  if (process.env.WEBPACK_DEV_SERVER_URL) {
    // Load the url of the dev server if in development mode
    win.loadURL(process.env.WEBPACK_DEV_SERVER_URL);
  } else {
    createProtocol('app');
    // Load the index.html when not in development
    win.loadURL('app://./index.html');
  }
```

主进程生命周期和各个事件的定义

- ipcMain自定义事件