# 模块

```bash
# 导入标准模块 所有
import module1[, module2[,... moduleN]

# 导入模块中的单个
from modname import name1[, name2[, ... nameN]]

# 导入模块的所有
from modname import *
```

## `__name__` 属性

一个模块被另一个程序第一次引入时，其主程序将运行。

```python
#!/usr/bin/python3
# Filename: using_name.py

if __name__ == '__main__':
   print('程序自身在运行')
else:
   print('我来自另一模块')
```

## 包

包是一种管理 Python 模块命名空间的形式，采用"点模块名称"。

```bash
sound/                          顶层包
      __init__.py               初始化 sound 包
      formats/                  文件格式转换子包
              __init__.py
              wavread.py
              wavwrite.py
              aiffread.py
              aiffwrite.py
              auread.py
              auwrite.py
              ...
      effects/                  声音效果子包
              __init__.py
              echo.py
              surround.py
              reverse.py
              ...
      filters/                  filters 子包
              __init__.py
              equalizer.py
              vocoder.py
              karaoke.py
              ...
```

- 在导入一个包的时候，Python 会根据 sys.path 中的目录来寻找这个包中包含的子目录。
- 目录只有包含一个叫做 __init__.py 的文件才会被认作是一个包，主要是为了避免一些滥俗的名字（比如叫做 string）不小心的影响搜索路径中的有效模块。
- 最简单的情况，放一个空的 :file:__init__.py就可以了。

注意事项： 因为Windows是一个大小写不区分的系统。

导入语句遵循如下规则：如果包定义文件 `__init__.py` 存在一个叫做 `__all__`的列表变量，那么在使用 `from package import *` 的时候就把这个列表中的所有名字作为包内容导入。

`__all__` 相当于需要导出的所有子模块的名称，只有使用到`from ... import *`的时候需要。

```python
# /sound/effects/__init__.py

#用到 * 的时候就要配置
__all__ = ["echo", "surround", "reverse"]
```

```bash
import sound.effects.echo
import sound.effects.surround
from sound.effects import *
```

主模块的名字永远是 `"__main__"`，一个 `Python`应用程序的主模块，应当总是使用绝对路径引用。

包还提供一个额外的属性 `__path__`。这是一个目录列表，里面每一个包含的目录都有为这个包服务的 `__init__.py`，你得在其他`__init__.py` 被执行前定义哦。可以修改这个变量，用来影响包含在包里面的模块和子包。

这个功能并不常用，一般用来扩展包里面的模块。