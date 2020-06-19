# [python - 1 - 基本概念](https://www.runoob.com/python3/python3-basic-syntax.html)

## 全局环境变量

环境变量是让操作系统更快查找你要查找的资源。包括模块的导入和导出。

- `PYTHONPATH`：PYTHONPATH 是 Python 搜索路径，默认我们`import`的模块都会从 PYTHONPATH 里面寻找。
- `PYTHONSTARTUP`：Python 启动后，先寻找 PYTHONSTARTUP 环境变量，然后执行此变量指定的文件中的代码。
- `PYTHONCASEOK`：加入 PYTHONCASEOK 的环境变量, 就会使 python 导入模块的时候不区分大小写.
- `PYTHONHOME`：另一种模块搜索路径。它通常内嵌于的`PYTHONSTARTUP`或`PYTHONPATH`目录中，使得两个模块库更容易切换。

## 三种注释 # ''' """

```python
# 这是注释

'''
这也是注释
'''

"""
这也是注释
"""
```

## 严格规范：行与缩进

python 最具特色的就是使用缩进来表示代码块，不需要使用大括号 {} 。

缩进的空格数是可变的，但是同一个代码块的语句必须包含相同的缩进空格数。

```python
if True:
    print ("Answer")
    print ("True")
else:
    print ("Answer")
  print ("False")    # 缩进不一致，会导致运行错误
```

## 多行语句

Python 通常是一行写完一条语句，但如果语句很长，我们可以使用反斜杠(\)来实现多行语句

```python
total = item_one + \
        item_two + \
        item_three
```

## 多个变量赋值

```python
# 传统的方式
a = b = c = 1

# 对称的方式
a, b, c = 1, 2, "runoob"
```

## 空行

函数之间或类的方法之间用空行分隔，表示一段新的代码的开始。类和函数入口之间也用一行空行分隔，以突出函数入口的开始。

空行与代码缩进不同，空行并不是 Python 语法的一部分。书写时不插入空行，Python 解释器运行也不会出错。但是空行的作用在于分隔两段不同功能或含义的代码，便于日后代码的维护或重构。

记住：空行也是程序代码的一部分。

## 同一行显示多条语句

Python 可以在同一行中使用多条语句，语句之间使用分号(;)分割。

```python
#!/usr/bin/python3

import sys; x = 'runoob'; sys.stdout.write(x + '\n');
```

## 等待用户输入 input

```python
#!/usr/bin/python3

input("\n\n按下 enter 键后退出。")

input("这是提示文本：请输入任意按钮，enter键退出")

input("请输入任意字符，然后回车：")
```

## Print 输出

`print` 默认输出是换行的，如果要实现不换行需要在变量末尾加上 `end=""`

```python
#!/usr/bin/python3

x="a"
y="b"
# 换行输出
print( x )
print( y )

print('---------')
# 不换行输出
print( x, end=" " )
print( y, end=" " )
print()
```

## import 与 from...import

在 `python` 用 `import` 或者 `from...import` 来导入相应的模块。

- 将整个模块(somemodule)导入，格式为： `import somemodule`
- 从某个模块中导入某个函数,格式为： `from somemodule import somefunction`
- 从某个模块中导入多个函数,格式为： `from somemodule import firstfunc, secondfunc, thirdfunc`
- 将某个模块中的全部函数导入，格式为： `from somemodule import *`

```python
import sys
print('================Python import mode==========================')
print ('命令行参数为:')
for i in sys.argv:
    print (i)
print ('\n python 路径为',sys.path)
```
