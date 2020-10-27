# pythone - 2 - 数据类型

基本数据类型：

- `Number`：数字
- `String`：字符串
- `List`：列表
- `Tuple`：元组
- `Set`：集合
- `Dictionary`：字典

## 区分`不可变数据`和`可变数据`

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

常量：

- `e`
- `pi`

强数据类型转化：

- `float`
- `int`
- `complex(x)`
- `complex(x, y)`

数学函数：

- `abs(x)`： 绝对值
- `ceil(x)`： 向上取整
- `cmp(x, y)`： 比较函数
- `exp(x)`： e的x次幂
- `fabs(x)`： 
- `floor(x)`： 向下取整
- `log(x)`
- `max()`
- `min()`
- `pow(x, y)`： `x**y`
- `round(x, n)`： 四舍五入， `n`代表保留的小数点后几位数
- `sqrt(x)`： 数字x的平方根

随机函数：

- `choice(seq)`： 从序列的元素中随机挑选一个元素。
- `randrange(start, stop, step = 1)`： 从指定范围内，按指定基数递增的`集合`中获取一个随机数。
- `random`： 随机生产一个实数，它在`[0, 1)`范围内。
- `seed(x)`： 改变随机数生成器的种子。
- `shuffle(lst)`： 将序列的所有元素随机排序。
- `uniform(x, y)`： 随机生成下一个实数，它在[x, y]范围内。

三角函数：

- `sin(x)`
- `cos(x)`
- `tan(x)`
- `degress(x)`： 将弧度装换成角度。
- `radians(x)`： 将角度转换成弧度。

注意事项：

- 除法 `/` 总是返回一个浮点数。可以考虑 `//` 但是会丢失分数部分，该操作符是向下取整。

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

字符串格式化：

```bash
print ("我叫 %s 今年 %d 岁!" % ('小明', 10))
```

三引号：

```bash
#!/usr/bin/python3
 
para_str = """这是一个多行字符串的实例
多行字符串可以使用制表符
TAB ( \t )。
也可以使用换行符 [ \n ]。
"""

print (para_str)
```

f-string: f-string 格式化字符串以 f 开头，后面跟着字符串，字符串中的表达式用大括号 {} 包起来，它会将变量或表达式计算后的值替换进去。

```bash
>>> name = 'Runoob'
>>> 'Hello %s' % name
'Hello Runoob'

# f-string： 插值
>>> name = 'Runoob'
>>> f'Hello {name}'  # 替换变量

>>> f'{1+2}'         # 使用表达式
'3'

>>> w = {'name': 'Runoob', 'url': 'www.runoob.com'}
>>> f'{w["name"]}: {w["url"]}'
'Runoob: www.runoob.com'
```

## List列表

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

列表函数&方法：

- `len(list)`： 列表元素个数
- `max(list)`： 返回列表元素最大值
- `min(list)`： 返回列表元素最小值
- `list(seq)`： 将元组转为列表
- `append`： 尾部添加元素
- `count`： 当个元素在列表中出现的次数
- `extend`： 尾部合并另一个列表
- `index`： `indexOf`
- `insert`： 将对象插入列表
- `pop`： 弹出尾部元素
- `remove`： 移除列表中某个值的第一个匹配项
- `reverse`： 反转列表
- `sort`： 排序
- `clear`： 清空列表
- `copy`： 复制列表

## Tuple（元组） 元组使用`()` 列表使用`[]`

元组（tuple）与列表类似，不同之处在于元组的元素不能修改。元组写在小括号 () 里，元素之间用逗号隔开。

```bash
>>>tup1 = ('Google', 'Runoob', 1997, 2000)
>>> tup2 = (1, 2, 3, 4, 5 )
>>> tup3 = "a", "b", "c", "d"   #  不需要括号也可以
>>> type(tup3)
<class 'tuple'>
```

内置函数：

- `min(tuple)`
- `max(tuple)`
- `len(tuple)`
- `tuple(iterable)`

## Set（集合）

集合（set）是由一个或数个形态各异的大小整体组成的，构成集合的事物或对象称作元素或是成员。基本功能是进行成员关系测试和删除重复元素。

Set comprehension：

```bash
>>>a = {x for x in 'abracadabra' if x not in 'abc'}
>>> a
{'r', 'd'}
```

## Dictionary（字典）

键必须是唯一的，但值则不必。

值可以取任何数据类型，但键必须是不可变的，如`字符串`，`数字`或`元组?`。

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

一些删除的操作

```pytho
#!/usr/bin/python3

dict = {'Name': 'Runoob', 'Age': 7, 'Class': 'First'}

del dict['Name'] # 删除键 'Name'
dict.clear()     # 清空字典
del dict         # 删除字典

print ("dict['Age']: ", dict['Age'])
print ("dict['School']: ", dict['School'])
```

自带函数：

- `clear()`
- `copy()`
- `formkeys()`
- `get(key, default = None)`
- `key in dict`
- `items()`
- `keys()`
- `setdefault(key, default = None)`
- `update(dict2)`
- `values()`
- `pop()`
- `popitem()`

## 数据类型转换

- `int(x, base?)`：将x转换为一个整数
- `float(x)`：将x转换到一个浮点数
- `complex(real, image?)`：创建一个复数
- `str(x)`：将对象x转换为字符串
- `repr(x)`：将对象x转换为表达式字符串
- `eval(x)`：用来计算在字符串中的有效Python表达式，并返回一个对象
- `tuple(s)`：将序列s转换为一个元组
- `list(s)`：将序列s转换为一个列表
- `set(s)`：转换为可变集合
- `dict(d)`：创建一个字典
- `frozenset(s)`：转换为不可变集合
- `chr(x)`：将一个整数转换为一个字符
- `ord(x)`：将一个字符转换为它的整数值
- `hex(x)`：将一个整数转换为一个十六进制字符串
- `oct(x)`：将一个整数转换为一个八进制字符串
