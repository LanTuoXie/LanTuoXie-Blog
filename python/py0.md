# python 学习记录

学习资料

- [菜鸟教程](https://www.runoob.com/python3/python3-tutorial.html)
- [Python 官方文档](https://docs.python.org/zh-cn/3/library/functions.html)
- [Python 简明教程](https://bop.mol.uno/11.modules.html)
- [Python Cookbook](https://python3-cookbook.readthedocs.io/zh_CN/latest/)
- [Python awesome-python-cn](https://github.com/jobbole/awesome-python-cn)

## 特殊的地方

- `变量` 只需被赋予某一值。不需要 `声明` 或定义 `数据类型`。
- 每一行 `物理行` 最多只写入一行 `逻辑行`。
- `空白区`（使用空格 `' '` 或制表符 `\t`）。
- 使用 `四个空格` 来缩进。这是来自 Python 语言官方的建议。好的编辑器会自动为你完成这一工作。请确保你在缩进中使用数量一致的空格，否则你的程序将不会运行，或引发不期望的行为。
- `+`： 代表拼接； `*`： 代表重复；
- `python： 函数的参数` `sorted(reverse = True)`。
- 字典的键很特殊, 键的数据类型只能是 `string` `number` `tuple`，但是字典的值可以是任意类型。`键的数据类型只能是不可变数据类型`。
- `Python` 中没有单独的 `char` 的数据类型。
- `单引号` 括起的字符串和 `双引号` 括起的字符串是一样的。
- `print` 总是会以一个不可见的 `新一行字符 \n` 结尾, 如果不想换行，可以通过 `end` 指定其应以空白结尾 `''`。
- `Python` 是强 `（Strongly）` 面向对象的，因为所有的一切都是对象， 包括数字、字符串与函数。
- `Python` 将优先计算表中位列于后的较高优先级的运算符与表达式。
- `Python` 中不存在 `switch` 语句。

```python
dict = { 'strType': 123 }
dict = { 0: 123 }
dict = { (123): 123 }
keyname = 'strKey'
dict = { keyname: 123 }
```

- 要区分字典和对象，字典是`python`的一个基本数据类型，而对象是类的实例。
- 要区分好`python`中的 `迭代器` `队列` `元组`，有些结构内部自带了 `迭代器特性`。
- `python`中没有`do ... while`循环语句。
- `pass`语句，用于占位，pass是空语句。
- 定义函数使用关键字 `def`。
- `Python` 中的 `pass` 语句用于指示一个没有内容的语句块。

## 字符串换行问题

如果想换行可以添加 `\n` ，如果不想换行可以添加 `\`。程序在默认行为中会自动换行，但在很多业务场景中，是不需要换行的。

```python
"This is the first sentence. \
This is the second sentence."
```

```python
"This is the first sentence. This is the second sentence."
```

## 超级常用内置函数

- `len`： 获取字符串或者列表的长度。
- `range`： 创建一个有序的数字序列。
- `dir:` 函数能够返回由对象所定义的名称列表。 如果这一对象是一个模块，则该列表会包括函数内所定义的函数、类与变量。
- `vars:`

## 作用域问题

`全局` `局部` `global`

## 函数中常用的概念

`def` `默认参数值` `关键字参数` `可变参数` `**dictionary` `*tuple` `DocStrings`

- `DocStrings:` 定义函数的第一行作为这个函数的文档说明
