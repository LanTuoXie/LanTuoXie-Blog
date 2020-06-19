# pythone - 2 - 数据类型

基本数据类型：

- `Number`：数字
- `String`：字符串
- `List`：列表
- `Tuple`：元组
- `Set`：集合
- `Dictionary`：字典

## 区分 `不可变数据` 和 `可变数据`

`不可变数据`： `Number` `String` `Tuple`

`可变数据`： `List` `Dictionary` `Set`

## 检测变量数据类型函数 `type()`

```python
>>> a, b, c, d = 20, 5.5, True, 4+3j
>>> print(type(a), type(b), type(c), type(d))
<class 'int'> <class 'float'> <class 'bool'> <class 'complex'>
```

## Number 数字类型 (4 种)

- `int整数`
- `bool布尔`： `True` `False`
- `float浮点数`： `1.23` `3E-2`
- `complex复数`： `1 + 2j` `1 + 2.2j`

## String 字符串类型

- `python`中单引号和双引号使用完全相同
- 使用三引号可以指定一个多行字符串`'''` `"""`
- 转义字符 `\`
- `r`可以让字符串中的`\`不执行转义的命令
- `+`字符串拼接，`*`字符串重复
- 字符串索引方式：从左往右以 0 开始，从右往左从-1 开始
- 一个字符就是长度为 1 的字符串
- 字符串截取的语法：`变量[头下标:尾下标:步长]`

```python
#!/usr/bin/python3

str='Runoob'

print(str)                 # 输出字符串
print(str[0:-1])           # 输出第一个到倒数第二个的所有字符
print(str[0])              # 输出字符串第一个字符
print(str[2:5])            # 输出从第三个开始到第五个的字符
print(str[2:])             # 输出从第三个开始后的所有字符
print(str * 2)             # 输出字符串两次
print(str + '你好')        # 连接字符串

print('------------------------------')

print('hello\nrunoob')      # 使用反斜杠(\)+n转义特殊字符
print(r'hello\nrunoob')     # 在字符串前面添加一个 r，表示原始字符串，不会发生转义
```

数值运算：

```python
>>> 5 + 4  # 加法
9
>>> 4.3 - 2 # 减法
2.3
>>> 3 * 7  # 乘法
21
>>> 2 / 4  # 除法，得到一个浮点数
0.5
>>> 2 // 4 # 除法，得到一个整数
0
>>> 17 % 3 # 取余
2
>>> 2 ** 5 # 乘方
32
```

## List 列表

```base
变量[头下标：尾下标]
```

- `+`：连接运算符
- `*`：重复操作

```python
#!/usr/bin/python3

list = [ 'abcd', 786 , 2.23, 'runoob', 70.2 ]
tinylist = [123, 'runoob']

print (list)            # 输出完整列表
print (list[0])         # 输出列表第一个元素
print (list[1:3])       # 从第二个开始输出到第三个元素
print (list[2:])        # 输出从第三个元素开始的所有元素
print (tinylist * 2)    # 输出两次列表
print (list + tinylist) # 连接列表
```

## Tuple（元组）

元组（tuple）与列表类似，不同之处在于元组的元素不能修改。元组写在小括号 () 里，元素之间用逗号隔开。

## Set（集合）

集合（set）是由一个或数个形态各异的大小整体组成的，构成集合的事物或对象称作元素或是成员。基本功能是进行成员关系测试和删除重复元素。

## Dictionary（字典）

列表是有序的对象集合，字典是无序的对象集合。两者之间的区别在于：字典当中的元素是通过键来存取的，而不是通过偏移存取。

字典是一种映射类型，字典用 `{ }` 标识，它是一个无序的 `键(key) : 值(value)` 的集合。

```python
#!/usr/bin/python3

dict = {}
dict['one'] = "1 - 菜鸟教程"
dict[2]     = "2 - 菜鸟工具"

tinydict = { 'name': 'runoob','code': 1, 'site': 'www.runoob.com' }


print (dict['one'])       # 输出键为 'one' 的值
print (dict[2])           # 输出键为 2 的值
print (tinydict)          # 输出完整的字典
print (tinydict.keys())   # 输出所有键
print (tinydict.values()) # 输出所有值
```

## 数据类型转换

- `int(x, base?)`：将 x 转换为一个整数
- `float(x)`：将 x 转换到一个浮点数
- `complex(real, image?)`：创建一个复数
- `str(x)`：将对象 x 转换为字符串
- `repr(x)`：将对象 x 转换为表达式字符串
- `eval(x)`：用来计算在字符串中的有效 Python 表达式，并返回一个对象
- `tuple(s)`：将序列 s 转换为一个元组
- `list(s)`：将序列 s 转换为一个列表
- `set(s)`：转换为可变集合
- `dict(d)`：创建一个字典
- `frozenset(s)`：转换为不可变集合
- `chr(x)`：将一个整数转换为一个字符
- `ord(x)`：将一个字符转换为它的整数值
- `hex(x)`：将一个整数转换为一个十六进制字符串
- `oct(x)`：将一个整数转换为一个八进制字符串
