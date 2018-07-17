## cnpm 莫名奇妙bug 莫名奇妙的痛


最近想搭建`react@v16` 和 `react-router@v4`，搭建过程打算用vue脚手架webpack模板那套配置方法(webpack3)。
由于我之前安装的是webpack4，和高版本的webpack-dev-server，Vue那个是webpack3。然后我就直接`cnpm i webpack@3.6.0 webpack-dev-server@2.9.1 -D`，本想着替换版本，然后运行也可以。

但是在我已经搭建好react babel esclint 后，发现修改组件，webpack是重新编译了，热更新也开启了，但是开发服务器没有刷新页面。
开发服务器要刷新页面，要接收到更新通知，可以重新编译后并没有通知服务器。然后我以为是配置出了问题，找了半天问题，心累。

然后各种找，翻看文档，最后打算切换成webpack4，和最新的webpack-dev-server版本试一试，还是不行。

long long ago ~~下班了，删了`node_modules`文件夹，重装`cnpm i`竟然可以了，我。。。

哈哈哈哈哈，欲哭无泪。
