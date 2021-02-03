# Electron构建和打包

electron开发比较困扰开发者的问题可能是包的安装和框架的选择。

[electron-vue-admin](https://github.com/PanJiaChen/electron-vue-admin) `git clone` 框架模板。

## 环境选择和安装

可以使用 `npm` 和 `yarn` 安装。

有些包需要翻墙，这个时候就要考虑到镜像。

`electron` 包在国内直接安装比较慢，可以考虑先使用`cnpm`安装或者直接`yarn`

但是最好还是你先设置镜像。

**npm包国内镜像源**

```bash
npm -----  https://registry.npmjs.org/
cnpm ----  http://r.cnpmjs.org/
taobao --  https://registry.npm.taobao.org/
nj ------  https://registry.nodejitsu.com/
rednpm -- http://registry.mirror.cqupt.edu.cn
skimdb -- https://skimdb.npmjs.com/registry
yarn ----  https://registry.yarnpkg.com
```

**electron设置国内镜像源**

```bash
npm config set electron_mirror https://npm.taobao.org/mirrors/electron/
```

**yarn设置国内镜像源**

```bash
yarn config set registry https://mirrors.huaweicloud.com/repository/npm/
yarn config set disturl https://mirrors.huaweicloud.com/nodejs/
yarn config set electron_mirror https://mirrors.huaweicloud.com/electron/

yarn config set registry https://registry.npm.taobao.org
yarn config set disturl https://npm.taobao.org/dist
yarn config set electron_mirror https://npm.taobao.org/mirrors/electron/
```

## 打包的坑

打包工具下载存放的位置：`C:\Users\Administrator\AppData\Local\electron-builder\Cache`

[手动安装打包必须工具](https://npm.taobao.org/mirrors/electron-builder-binaries/)

- 如果工具无法下载第一个包`xxx.zip`，可以试试`electron_mirror https://mirrors.huaweicloud.com/electron/` 华为的镜像，下载挺快的。
- 其它包可以在上方淘宝镜像中手动下载，然后安装到`C:\Users\Administrator\AppData\Local\electron-builder\Cache`下方。
- `unpack包`打包好后。



**打包之前需要注意的事项：**

- 先跑 `npm run build:clean`。
- 然后把`dist/electron`的文件清空，如果打包工具没有安装。
- 不要把`build/icons`删了，icons被删打包会失败。