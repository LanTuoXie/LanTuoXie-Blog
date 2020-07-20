# git 基本操作和指令

记录`git`的一些基本指令和概念。

## git 和 svn （分布式和集中式）

- 分布式版本控制系统根本没有“中央服务器”，每个人的电脑上都是一个完整的版本库。
- 集中式版本控制系统，版本库是集中存放在中央服务器的，而干活的时候，用的都是自己的电脑，所以要先从中央服务器取得最新的版本，然后开始干活，干完活了，再把自己的活推送给中央服务器。

## git 配置

```bash
# 本地生成一个版本库 .git 版本库
git init

# 获取配置 vscode 自动用默认浏览器打开 config 文档
git config --help

# 获取git 基本配置信息可以使用下面指令
git config --list

# git 的配置有系统、 全局和局部之分
git config user.email xxx@xx.com --local
git config user.email xxx@xx.com --global
git config user.email xxx@xx.com --system

# 一般 user 需要配置 2 个配置 emial 和 name
git config user.email xxx@xx.com --local
git config user.name xxxx --local
```

## .gitignore

```bash
# 该文件下的文件会被git过滤掉，也即不会被添加到本地库以及提交到远程库
# 查看文件是否忽略了，可以用下面的指令
git status
```

## 如果是初建立的项目

```bash
# 先克隆项目，克隆操作不用什么操作
git clone [<https协议地址> | <ssh协议地址>]

# 配置远程仓库的地址 remote
git remote --help
git remote add origin [<https协议地址> | <ssh协议地址>]

# 查看所有的 remote，该指令会列出 远程地址的变量
git remote

# 将本地库修改的文件添加到暂存库
# 将当前目录下所有文件添加到暂存库
git add .
git add filename.txt vuefilename.vue

#
```

## 更新 fork 项目

```bash
# redux: https://github.com/reduxjs/redux.git
# fork redux: https://github.com/LanTuoXie/redux.git

# 先将 fork 的项目克隆下来
git clone https://github.com/LanTuoXie/redux.git

# 创建另一个 remote
git remote add upstream https://github.com/reduxjs/redux.git

# 将最新的 upstream remote 拉取到本地
git fetch upstream

# 合并 origin remote 和 upstream remote
# 一般 clone 下来的 remote 是 origin
git merge upstream/master

# 将合并后的结果提交到 origin remote 的 master 分支
git push origin master
```

## 引用

- [廖雪峰 git 教程](https://www.liaoxuefeng.com/wiki/896043488029600/896202780297248)
