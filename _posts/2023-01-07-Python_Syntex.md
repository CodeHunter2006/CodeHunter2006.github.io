---
layout: post
title: "Python 语法"
date: 2023-01-07 22:00:00 +0800
tags: Python
---

记录 Python 特殊语法点

# 基本格式

- 用空格缩进作为"代码块"的标志，必须严格遵守，同一缩进下的代码被认为是顺序关系
- 一行的多个表达式可以用`;`隔开，从而执行多个语句

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

### 字符串运算符

- `+`
  字符串连接
- `*n`
  重复输出字符串`"hello"*2  # hellohello`
- `[n]`
  用索引获取单个字符
- `[a:b]`
  用下标以"左闭右开"截取子字符串
- `in`
  如果字符串中包含给定的字符则返回`True`，如：`'H' in "Hello" # True`
- `not in`
  与上面相反
- `r/R`
  以这个符号开头的字符串不做任何转义处理
- `%`
  相当于`format`函数，如：`"H%sllo" % "e" # Hello`

## dict

- `**`紧贴 dict 变量前面，如`func1(**dict1)`，表示把 dict 以`key1=value1, ...`的形式展开。通常作为一个函数的参数使用。

### dict 内置函数

- `dict.items()`返回一个 dict 内容的 list，形如：`[(key1,value1), (key2,value2), ...]`

## set

集合也是一种字典，只不过只有 key 没有 value

```python
set1 = {1,2,3}
```

## tuple(元组)

相当于一个固定内容的数组，数组内容无法修改，并且元素类型可以不同。

```python
x = (1, "2", {3})
```

### 元组内置函数

- `len(tuple)` 计算元素个数
- `max(tuple)` 返回最大值
- `min(tuple)` 返回最小值
- `tuple(iterable)` 将可迭代序列转为元组

## None

`None`在 Python 中是一个对象，有其特定类型 NoneType。
如果一个变量未定义，则会直接报错，不会允许继续执行下去，这和 Javascript 的 undefine 是不同的。

创建一个变量但不确定类型和值时，可以用`x = None`

## enum

- Python 的 enum 类型可以借助`enum.Enum`实现三个关键功能：
  - 不允许重复定义枚举值，会报错
  - 外部不可以修改枚举值，会报错
  - 枚举值应对应一个 int 数值或其它类型值

```python
from enum import Enum
class Weekday(Enum):
    monday = 1
    tuesday = 2
    wednesday = 3
    thirsday = 4
    friday = 5
    saturday = 6
    sunday = 7
    test = "test" # 对应的 value 也可以是 int 之外的类型

print(Weekday.wednesday)         # Weekday.wednesday
print(type(Weekday.wednesday))   # <enum 'Weekday'>
print(Weekday.wednesday.name)    # wednesday
print(Weekday.wednesday.value)   # 3
```

## 预期类型元数据(type hints)

Python 作为一个脚本是不支持参数和返回值设置类型的，但是为了减少类型匹配问题，Python3.3 增加了`-> type`和`xx: type`语法，为函数返回值或参数设置预期类型，供开发者和工具参考。不过仅供参考，运行时不做类型检查。

```python
# 参数类型用 : type 表示
# 返回值类型用 -> type 表示
# 还可以设置默认值
def func1(param1: int, param2: str = "default") -> dict {
}
```

- 类型要用特殊类型名表达，不可以用`literal`表达

  - 比如`(int,int)`正确的表达是`typing.Tuple[int, int]`
    - 注意，是`typing`包下的类，参数要以`[]`
  - 可用类型参考：[typing — Support for type hints](https://docs.python.org/3/library/typing.html)

- `callable`
  可以用来作为类型，表示可被调用的函数。也可以用来做类型判断函数，如果是函数则返回`True`

# 逻辑控制

## pass

python 的 if 等逻辑控制语句后不允许什么都不做，所以引入了 pass，作为占位语句。

```python
if xx:
  pass

def func1:
  pass

```

- 用三个下点`...`和`pass`的作用完全相同

## if

- 在 Python 的逻辑表达式中，`None`、`False`、`0`、`""(空字符串)`、`[](空列表)`、`()(空元组)`、`{}(空字典)`都相当于`False`。

- 如何判断变量值是否为`None`？
  - `if X is None:`
  - `if X is not None:`相当于`if not (X is None)`

## 逻辑运算符

- `and or not`

## 比较运算符

- Python 允许多个`==`连续相等比较，如果都相等则成立，如 `if a == b == c == d:`

- `obj1 is obj2`
  判断两个对象指针是否指向同一对象

## while

```python

while condition:
  pass
else: # else 是可选的，表示退出 while 时执行一次
  pass
```

- 感觉这`else`完全没用啊，while 执行完毕继续执行的代码就是只执行一次

## for

for 用来对容器进行迭代

```python
for item in collection:
    print(item)
    pass
else: # 同样可选
    pass

# 如果每个元素是个 tuple，则可以用多个字段承接元素
for v1, v2, v3 in tuple_collection:
  pass

# 可以利用 items 函数将 dict{k:v} 转为 list[(k,v)...]
for k, v in d.items():
  pass
```

## continue, break

- continue 和 break 的基本用法和其他语言一样
- Python 不支持双层循环中，直接操作外层循环

- **注意**如果循环包含`else`，那么`break`将导致`else`失效。`continue`不会影响`else`。

## 三目运算符

Python 中支持三目运算语法：

`[statement_1] if [expression] else [statement_2]`
如果`expression`为`True`则返回`statement_1`，否则返回`statement_2`

## Comprehension(推导式)

Python 的推导式语法允许从一种集合导出另一种集合，这里的"集合"不是狭义的 set，而是广义的 container。

- 列表推导式

  - `[out_exp_res for out_exp in input_list [if condition with out_exp]]`
    把符合条件的元素导出到新列表，如果没有条件则全部导入

- 字典推导式

  - `{ key_expr: value_expr for value in collection [if condition] }`
    字典推导式的数据源只能是更低维度的类型，比如 list set tuple
  - 可以利用`dict.items()`作为数据源进行推导
    `{ key_expr: value_expr for (k,v) in dict_src [if condition] }`

## 拉姆达表达式 lambda

- 注意，拉姆达不是函数，只是一个表达式

`lambda [arg1][,arg2]: expression`

例：`cmp = lambda x, y: x >= y`

## 迭代器

迭代器是一个对象，可以被`next(it)`不断调用返回结果。

```python
list=[1,2,3]
it = iter(list)  # 用内置的 iter 函数创建迭代器
print(next(it)) # 1
print(next(it)) # 2
print(next(it)) # 3
print(next(it)) # exception: StopIteration

it = iter(list)
for x in it:  # 可以用 for 迭代
    print (x, end=" ")  # 1 2 3

# 自定义可迭代类
class MyNumbers:
  max = 0
  a = 0
  def __init__(self, max):
    self.max = max

  def __iter__(self):
    self.a = 0
    return self

  def __next__(self):
    if self.a <= self.max:
      x = self.a
      self.a += 1
      return x
    else:
      raise StopIteration # 用 StopIteration exception 表示结束

myObj = MyNumbers(2)
myiter = iter(myObj)

for x in myiter:
    print(x, end=" ") # 0 1 2
```

## yield

使用了 yield 的函数被称为**生成器**，其中 yield 类似 return，但是再次执行该函数时，逻辑将从 yield 下一行继续执行。

```python
def _generator(times: int) -> int:
    x = 0
    while x < times:
        yield x # 这里返回了结果，但是调用栈转到下一行、暂停
        print("go on " + str(x))
        x += 1

def testYield():
    iter = _generator(3)  # 生成器执行后产生一个迭代器
    for item in iter:
        print(item)
testYield()
#0
#go on 0
#1
#go on 1
#2
#go on 2
```

# 函数

## 传参，传引用、传值

- 定义函数语法：`def func_name(param1 [: type] [= init_value], param2) [-> result_type]:`

- 如果函数要求参数，但是调用的地方没传参，在调用处会报错

- 在 python 中，`strings`, `tuples`, 和 `numbers` 是不可更改(immutable)的对象，而 `list`, `dict` 等则是可更改(mutable)的对象。
  不可更改的对象按"值"传参，即函数中的变量实际上是新的变量，函数中的修改不会影响函数外变量；可更改的对象按"引用"传参，可以修改外部变量。

## 不定长参数

- 在函数定义时，最右边可以带一个不定长参数`def functionname([formal_args,] *var_args_tuple ):`，这里`var_args_tuple`是一个 tuple，包含了所有后面未命名的参数的值

## 参数默认值

- 可以赋默认值：`def test(param1: str = '')`

# 变量

- 用`_`开头的变量可以避免被导出

- Python 有类似 Go 的**并列赋值**语法: `a, b, c = 1, "2", (3)`

# 异常处理

Python 的异常处理与 Java 基本相同。

# module(模块)

module 是 python 代码重用的的基本单元，一个 module 就是一个`xx.py`文件，这里的`xx`就是模块名。

- 使用`import xx`导入模块名，具体元素需要`xx.a`调用
- 使用`from xx import *`导入模块内的所有全局变量/函数
- 使用`from xx import a, b`精确导入所需元素
- 使用`from ..aa.bb import c`可以导入上层 package 的子 package，这里两个点`..`表示上一层
- 使用`from ...aa.bb import c`可以一直向上找到某个 package，其子 package 包含`aa.bb`

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

- 作用域就是一个变量名对应的查找范围，从近到远分为 4 层：L(Local)->E(Enclosing)->G(Global)->B(Built-in)
- 只有模块（module），类（class）以及函数（def、lambda）才会引入新的作用域，其它的代码块（如 if/elif/else/、try/except、for/while 等）是不会引入新的作用域的，也就是 if 中定义的新变量其实直接定义在 Local 作用域中了，在 if 外层也是可以访问的。

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

Python 的面向对象支持多继承。

- 多继承时`X(A,B,C)`如果有相同方法名而子类中未指定，则从左向右广度优先遍历。

```python
class A:
  pass

class B:
  pass

class C(A, B): # C 继承自 A 和 B
  a = 1 # 私有成员
  __b = 2 # 用双下划线开头实现私有变量

  def __init__(self, paramA: int, paramB = 3: int): # 构造函数
    A.__init__()  # 指定父类方法
    self.a = paramA
    self.__b = paramB

  def f(self): # 成员函数
    # 这里的 self 相当于 this 指针
    print("f " + str(self.a))

c = C(2)  # 调用构造函数
c.f()  # f 2
```

- 判断是否是实例`isinstance(object, class)`
  子孙类对象也会返回`True`

- 类的专有方法：
  - `__init__` : 构造函数，在生成对象时调用
  - `__del__` : 析构函数，释放对象时使用
  - `__repr__` : 打印，转换
  - `__setitem__` : 按照索引赋值
  - `__getitem__`: 按照索引获取值
  - `__len__`: 获得长度
  - `__cmp__`: 比较运算
  - `__call__`: 函数调用
  - `__add__`: 加运算
  - `__sub__`: 减运算
  - `__mul__`: 乘运算
  - `__truediv__`: 除运算
  - `__mod__`: 求余运算
  - `__pow__`: 乘方

## 运算符重载

```python
#!/usr/bin/python3

class Vector:
   def __init__(self, a, b):
      self.a = a
      self.b = b

   def __str__(self):
      return 'Vector (%d, %d)' % (self.a, self.b)

   def __add__(self,other):
      return Vector(self.a + other.a, self.b + other.b)

v1 = Vector(2,10)
v2 = Vector(5,-2)
print (v1 + v2) # Vector (7, 8)
```

## `@classmethod`

相当于声明 class 中的一个方法为类的静态方法，该函数的第一个参数为这个类本身，这个参数可以用来执行`__init__`构造函数

```python
class A:
  field = 0 # 对象的成员变量或类的成员变量

  def __init__(self, x):
        self.field = x

  @classmethod
  def cls_f(cls, y):
    obj1 = cls(y) # 可以用 cls 表示对象构造函数
    cls.field = y
    return obj1

o = A.cls_f(1)
print(o.field, A.field) # 1 1
o = A.cls_f(1)
print(o.field, A.field) # 1 2
```

- 如果经过了构造函数，创建了新对象，则成员变量是**对象的成员变量**
- 如果直接通过类名调用，则成员变量是**类的成员变量**

# 常用函数

- `dir(SampleClass)`
  查看类的成员