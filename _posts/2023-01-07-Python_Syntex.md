---
layout: post
title: "Python 语法"
date: 2023-01-07 22:00:00 +0800
tags: Python
---

记录 Python 特殊语法点

# 注释

```python
# normal comment

"""
multiline
comment
"""

'''
multiline
comment
too
'''
```

# 类型

## string

### string 表示方法

```python
str1 = "string with escape character like \n"

str2 = 'also a string'

str3 = """
multiline
string
with out escape character
"""
```

- Python 不支持单字符类型，单字符在 Python 中也是作为一个字符串使用。

- 可以用`变量[头下标:尾下标]`截取子字符串。`0`为开始值，`-1`为最后一个字符，以此类推。

### bytes 与 str

str 是字符串，通常用`""`或`''`构造，底层编码为 Unicode 字符(0-65535)
bytes 是二进制字节数组，用`b''`构造，底层存储字节(0-255)

- str 转 bytes: `b = s.encode('utf-8')`
- bytes 转 str: `s = b.decode('utf-8')`

## dict

- `**`紧贴 dict 变量前面，如`func1(**dict1)`，表示把 dict 以`key1=value1, ...`的形式展开。通常作为一个函数的参数使用。

## 预期类型元数据

Python 作为一个脚本是不支持参数和返回值设置类型的，但是为了减少类型匹配问题，Python3.3 增加了`-> type`和`xx: type`语法，为函数返回值或参数设置预期类型，供开发者和工具参考。不过仅供参考，运行时不做类型检查。

```python
def func1(param1: int, param2: str) -> dict {
}
```

# 函数

## 传参，传引用、传值

- 在 python 中，`strings`, `tuples`, 和 `numbers` 是不可更改(immutable)的对象，而 `list`, `dict` 等则是可更改(mutable)的对象。
  不可更改的对象按"值"传参，即函数中的变量实际上是新的变量，函数中的修改不会影响函数外变量；可更改的对象按"引用"传参，可以修改外部变量。

## 不定长参数

- 在函数定义时，最右边可以带一个不定长参数`def functionname([formal_args,] *var_args_tuple ):`，这里`var_args_tuple`是一个 tuple，包含了所有后面未命名的参数的值

# 变量

- 用`_`开头的变量可以避免被导出

# 异常处理

Python 的异常处理与 Java 基本相同。

# module(模块)

module 是 python 代码重用的的基本单元，一个 module 就是一个`xx.py`文件，这里的`xx`就是模块名。

- 使用`import xx`导入模块名，具体元素需要`xx.a`调用
- 使用`from xx import *`导入模块内的所有全局变量/函数
- 使用`from xx import a, b`精确导入所需元素

# package(包)

module 要用包来组织，包对应于文件夹，文件夹名就是包名。包和文件夹一样，可以有嵌套结构，通过`.`分隔。

- 使用`import pa.pb.xx` 来导入包
- 在文件夹中放入`__init__.py`表示这个文件夹是一个包，里面的代码会在包第一次被加载时执行
- 在`__init__.py`可以设置变量`__all__ = ["echo", "surround", "reverse"]`如果用`from xxx import *`的导入方式，则会只导入`__all__`指定的元素
- 模块如果被间接导入多次，只会被加载一次，`__init__.py`只会被调用一次
- 如果多个模块都在一个包中，并且这些模块依赖相同的外部包，就可以把这些 import 语句放入`__init__.py`中，避免重复

## 在文件中区分导入和脚本执行

在`import xxx`和`python xxx.py`时都会执行脚本中的代码，但有些代码只想在被作为脚本时执行，比如`main()`函数，可以用下面方式区分：

```python
# def 语句总会被执行
def main():
	pass

if __name__ == '__main__':
	# 这里的 main() 只有非 import 执行时才会调用
	main()
```

# 作用域

- 在`.py`中定义的变量是全局变量，但是在函数中要使用时要用`global xx`声明一下，否则函数中的变量为局部变量

- 在函数内定义函数形成闭包时，可以在内部函数中用`nonlocal xx`声明该变量为上层函数中的变量，否则默认为局部变量

- 关于下划线`_`和作用域的关系
  - 在 class 中，函数名以双下划线`__`开头，则该函数由于 Python 的重命名机制，会自动在外部和子类隐藏，相当于其他语言`private`的作用
  - 在变量或函数名前加一个下划线`_`表示一种命名规范，这种变量或函数不推荐在外部使用或子类重写，应该被当做`private`看待，但是实际上是可以访问到的
  - 在变量后面加下划线，通常用于避免和关键字冲突，例如：`int_`
  - 以双下滑线开头和结尾的函数，如`__init__`，叫做**魔法方法**，是 Python 为各个类设置的默认函数，例如`l=[1,2,3]; len(l)`这里的`len`会被自动替换为`__len__(l)`函数。我们可以通过重写的方式重写底层函数逻辑，魔法方法大致分为下面类型：
    - 构造与初始化
    - 类的表示
    - 访问控制
    - 比较、运算等操作
    - 容器类操作
    - 可调用对象
    - 序列化

# 面向对象

## `@classmethod`

相当于声明 class 中的一个方法为类的静态方法，该函数的第一个参数为这个类本身，这个参数可以用来执行`__init__`构造函数

# 常用函数

- `dir(SampleClass)`
  查看类的成员
