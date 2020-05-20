# yarn 基本操作

记录一些`yarn`常用指令。

## 常用指令

安装项目的全部依赖

```bash
yarn install
```

移除依赖包

```bash
yarn remove [package]
```

升级依赖包

```bash
yarn upgrade [package]
yarn upgrade [package]@[version]
yarn upgrade [package]@[tag]
```

分别添加到`devDependencies` `peerDependencies` `optionalDependencies`

```bash
yarn add [package] --dev
yarn add [package] --peer
yarn add [package] --optional
```
