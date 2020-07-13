# 迭代器和生成器

- 迭代是 `Python` 最强大的功能之一，是访问集合元素的一种方式。
- 迭代器是一个可以记住遍历的位置的对象。
- 迭代器对象从集合的第一个元素开始访问，直到所有的元素被访问完结束。迭代器只能往前不会后退
- 字符串，列表或元组对象都可用于创建迭代器。
- 迭代器有两个基本的方法： `iter()` 和 `next()`。
- 把一个类作为一个迭代器使用需要在类中实现两个方法 `__iter__()` 与 `__next__()` 。
类的内部有个构造函数 `__init__()`。
- `__iter__()` 方法返回一个特殊的迭代器对象， 这个迭代器对象实现了 `__next__()`方法并通过 `StopIteration` 异常标识迭代的完成。
- `__next__()` 方法（Python 2 里是 `next()`）会返回下一个迭代器对象。

```python
class MyNumbers:
  def __iter__(self):
    self.a = 1
    return self
 
  def __next__(self):
    if self.a <= 20:
      x = self.a
      self.a += 1
      return x
    else:
      raise StopIteration

```

## 迭代器 `iter()` `next()`

- `iter()`： 是将列表 `list` 转化为迭代器
- `next()`： 是执行迭代器
- `StopIteration`： 是迭代器的完成点标识

## 生成器 

- 在 Python 中，使用了 `yield` 的函数被称为生成器 `generator`。更简单点理解生成器就是一个迭代器。调用一个生成器函数，返回的是一个迭代器对象。

- 在调用生成器运行的过程中，每次遇到 `yield` 时函数会暂停并保存当前所有的运行信息，返回 `yield` 的值, 并在下一次执行 `next()` 方法时从当前位置继续运行。
