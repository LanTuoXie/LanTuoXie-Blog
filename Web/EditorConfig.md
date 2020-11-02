# [EditorConfig](http://editorconfig.org/#overview)

## .editorconfig文件

在很多开源项目中，会出现这个文件，这个文件有何作用？</br>
`editorconfig` 帮助开发者的(编辑器和IDEs)**定义和维护**编程风格。</br>
有些编辑器不用安装插件，会自动识别`.editorconfig`文件，然后会按文件中的规范设置编程风格 </br>

**不用安装插件的:**

- `GitHub`
- `IntelliJIDEA`
- `VisualStudio`
- `WebStorm`

等等，其他的可以在[EditorConfig](http://editorconfig.org/#overview)查看

**要安装插件:**

- `ATOM`
- `eclipse`
- `Sublime Text`
- `Notepad++`
- `vsCode`

等等，其他的可以在[EditorConfig](http://editorconfig.org/#overview)查看 </br>
安装插件教程也可以在官网查看

## 文件匹配规则

- `*` : 匹配任何**字符串**，除了路径分隔符**/**  
- `**` : 匹配任何**字符串**
- `?` : 匹配任何**单个**字符
- `[name]` : 类似正则的 `/name/`
- `[!name]` : 除了`[name]`之外
- `{s1,s2,s3}` : 类似正则的`s1|s2|s3`，表示或
- `{1..9}` : 表示匹配的次数在1~9之间都可以
- `#` ： 表示注释

## 支持的编程风格属性

- `indent_style` ： 缩进风格，可能值为`tab`(Tab键) 和 `space`(空格键)，一般敲了`{}`我们要在中间再敲enter键，会自动缩进，这个决定是用tab还是space
- `indent_size` : 主要决定space次数，一般风格推荐2或者4
- `tab_width` : 使用tab风格时，一般不用设置，默认值为`indent_size`
- `charset` : 编码，可能值`latin1` `utf-8` `utf-8-bom` `utf-16be` `utf-16le`
- `trim_trailing_whitespace` : `true` or `false` 是否将行尾空格删除
- `insert_final_newline` : `true` or `false` 确保文件最后一行是一个空行
- `root` : `true` or `false` 是否以`.editorconfig`文件为根目录，使用**文件匹配**时会以这个为**开始相对路径**

## webpack文件为例

```bash
#[采用的相对路径以这个文件开始]
root = true

#匹配所有文件
#缩进采用tab、大小为4、编码utf-8、删除行尾空格、最后一行是空白行 、文件最大行223行
[*]
indent_style = tab
indent_size = 4
charset = utf-8
trim_trailing_whitespace = true
insert_final_newline = true
max_line_length = 233

#.prettierrc采用的规则
[.prettierrc]
indent_style = space
indent_size = 2

#.yml .yaml .json 文件采用的规则
[*.{yml,yaml,json}]
indent_style = space
indent_size = 2

#由于root=true ，test文件夹和.editorconfig文件应在同一文件夹内
#test/cases/parsing/bom/ 文件下的bomfile.css bomfile.js
[test/cases/parsing/bom/bomfile.{css,js}]
charset = utf-8-bom

#所有markdown文件的规则
[*.md]
trim_trailing_whitespace = false
```
