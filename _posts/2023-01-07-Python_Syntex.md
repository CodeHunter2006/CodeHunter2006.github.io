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
- 如果一行写不下跨行，则用`\`在上一行结尾

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

# 用 () 将多个字符串包裹，即使换行也能自动连接，不添加换行符
str4 = ('another'
        'multiline'
        'example' 'last')
print(str4) # anothermultilineexamplelast

# f-string, f字符串
# 在字符串前加'f/F'表示一个字符串自动添加format函数，用{xx}做字符串替换
# 用 { 和 } 做大括号转义
str4 = f"str4 {str2} bracket: {{, another bracket: }} "
```

- Python 不支持单字符类型，单字符在 Python 中也是作为一个字符串使用。

- 可以用`变量[头下标:尾下标]`**左闭右开**区间截取子字符串。`0`为开始值，`-1`为最后一个字符，以此类推。
  - 如果下标超过合理范围，则会自动修正到合理范围，如果无法修正，则返回空 list。不会报错

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

### 常用函数

```python
# 用字符串 list 生成新的字符串，由指定字符串分割
','.join(['1','2']) # "1,2"
```

## Number

Python 支持三种数字类型：int float complex

- `print(1/2) # 0.5`
  两个整型相除时，会自动转为浮点型

## bool

- string -> bool 转换时的坑：`bool("False")`的结果是`True`，想要设置为`False`要用空字符串

## dict

- `**`紧贴 dict 变量前面，如`func1(**dict1)`，表示把 dict 以`key1=value1, ...`的形式展开。通常作为一个函数的参数使用。

### dict 内置函数

- `dict.items()`
  返回一个 dict 内容的 list，形如：`[(key1,value1), (key2,value2), ...]`

- `dict.copy()`
  返回浅复制对象
- `dict.get(key, default=None)`
  返回指定 key 的值，如果不存在，返回 default 值
- `dict.pop(key[,default])`
  删除对应的 key 的 item，并返回对应的 value。
  - 可以设定 default 值，如果没有 key 则返回 defualt value
  - 如果没有这个元素且没有设定 defualt 值，则报错"key 不存在"
- `dict.update(dict2)`
  把 dict2 里的 key-value 覆盖到 dict，dict 特有的 key 保持不变。

- `del testDict['name']`
  删除一个元素

## set

集合也是一种字典，只不过只有 key 没有 value

```python
set1 = {1,2,3}
```

- 用`{}`创建一个空集合时，其实默认创建了 dict 类型。用`set()`创建空集合更合适。

- set 的"交/并/差"操作

```python
s1 = set([1,2])
s2 = set([2,3])
s3 = set([1,2,3])

# 交集
s1 & s2 # {2}

# 并集
s1 | s2 # {1,2,3}

# 差集
s1 - s2 # {1}
s2 - s1 # {3}

# 判断子集
s1 >= s2 # False, s1 包含或等于 s2
s3 > s1 # True, s3 包含 s1
```

### set 内置函数

- `set.add(x)`
  添加一个元素

## tuple(元组)

相当于一个固定内容的数组，数组内容无法修改，并且元素类型可以不同。

```python
x = (1, "2", {3})
```

- 在赋值时，可以自动实现`unpack`，即把一个 tuple 对象分解成对应数量的变量赋值，如：`x, y, z = (1, 2, 3)`

### 元组内置函数

- `len(tuple)` 计算元素个数
- `max(tuple)` 返回最大值
- `min(tuple)` 返回最小值
- `tuple(iterable)` 将可迭代序列转为元组

### 单元素元组

- 单元素元组用这种方式表达：`(10, )` 注意多了个`,`
  - 由于`(10)`既可以表示一个元组，也可以表示数学中的一个数字(括号被视为优先级标志)

## None

`None`在 Python 中是一个对象，有其特定类型 NoneType。
如果一个变量未定义，则会直接报错，不会允许继续执行下去，这和 Javascript 的 undefine 是不同的。

创建一个变量但不确定类型和值时，可以用`x = None`

## 各类型通用函数

- `len(x)`
  可以对 dict/set/list/tuple 进行计算，返回元素个数

- `+`
  - 数字相加
  - 字符串拼接
  - list 拼接

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

- 遇到 json 序列化报错时`TypeError: Object of type 'eekday' is not JSON serializable`，
  可以用多继承解决：`class Weekday(int, Enum):`

- 使用`package pymysql`时可能遇到`object has no attribute 'translate'`，因为不识别 Enum 类型，需要强转成 int 型使用

- 为了避免上面两个报错，可以不继承任何类创建`Weekday`类，但是无法保证"不重复"和"不可修改"特性，需要在使用中注意。

## 预期类型元数据(type hints)

Python 作为一个脚本是不支持参数和返回值设置类型的，但是为了减少类型匹配问题，Python3.3 增加了`-> type`和`xx: type`语法，为函数返回值或参数设置预期类型，供开发者和工具参考。不过仅供参考，运行时不做类型检查。

```python
# 变量类型用 : type 表示
# 参数类型用 : type 表示
# 返回值类型用 -> type 表示
# 还可以设置默认值
def func1(param1: int, param2: str = "default") -> dict {
  var1: int = param1
}
```

- 类型要用特殊类型名表达，不可以用`literal`表达

  - 可用类型参考：[typing — Support for type hints](https://docs.python.org/3/library/typing.html)
  - 比如`(int,int)`正确的表达是`typing.Tuple[int, int]`
    - 注意，是`typing`包下的类，并且参数要以`[]`包裹
  - 比如**嵌套**类型`typing.List[typing.Tuple[str, str]]`
  - 比如有**不定项** tuple`Tuple[int, ...]`
  - **可能返回 None** `Optional[str]`
  - 注意，这里的`typing.Dict`等**只能用于类型描述**，不可以真的当成类型去实例化对象，如`x = typing.Dict()`将会报错

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

- python 没有 `switch` 语法，只能用`elif`实现

## 逻辑运算符

- `and or not`

- `not`除了判断`bool/None`之外，还可以判断`string/int/float/dict/set/list/tuple`为零值或空情况。

```python
# 以下全为 True
not ""
not 0
not 0.0
not {}
not []
not ()
```

- `or` 除了做"逻辑或"的 bool 表达之外，还可以根据左右两边的表达式是否为 None，返回非 None 结果
  ```Python
  x = 1
  y = None
  print(x or y)  # 1
  ```

## 比较运算符

- Python 允许多个`==`连续相等比较，如果都相等则成立，如 `if a == b == c == d:`

- `obj1 is obj2`
  判断两个对象指针是否指向同一对象

## in 成员判断运算符

in 可以用于 str, list, set, dict(key), tuple 的成员判断

```python
a = 1
b = (1,2,3)
c = [1,2,3]
d = {1:"a",2:"b",c:"3"}
e = {1,2,3}
f = "123"

# 下面判断都返回 True
if a in b...
```

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

### List Comprehensions(列表生成式)

```python
test = [1,2]
testStr = [str(x) for x in test if True]  # 生成为字符串数组
```

## continue, break

- continue 和 break 的基本用法和其他语言一样
- Python 不支持双层循环中，直接操作外层循环

- **注意**如果循环包含`else`，那么`break`将导致`else`失效。`continue`不会影响`else`。

## with

对于系统资源如文件、数据库连接、socket 而言，使用之后必须要关闭，`with`可以很好的实现这个逻辑。

```python
with expression [as variable]:
    #with-block

with open(file, "w") as f:
    f.write("hello python")
```

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
  - 可以利用`dict.items()`(返回`[(k,v)]`)作为数据源进行推导
    `{ key_expr: value_expr for (k,v) in dict_src.items() [if condition] }`

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

## 关键字参数

- 传参顺序与定义顺序不一致时，可以使用关键字传参
- 关键字传参一定要放在普通参数的后边(右边)，如果关键字参数重复设置了前面的参数则会报错

```python
def f(x: int, y: str):
  pass

f(y = "y", x = 1)
```

## 声明 ※ 参数

- 声明函数时可以用`*`作为参数分割关键字参数，要求调用传参时`*`后面的必须用关键字参数

```python
def func(x, y, *, z):
  pass

func(1, 3, z=3)
```

## 不定长参数

- 在函数定义时，最右边可以带一个不定长参数`def functionname([formal_args,] *var_args_tuple ):`，这里`var_args_tuple`是一个 tuple，包含了所有后面未命名的参数的值

- `def functionname([formal_args,] **var_args_dict):`，这里`var_args_dict`是一个 dict，包含了后面所有关键字参数，但是如果调用方只传了一个参数，则这里`var_args_dict`并不一定是一个 dict，而是与这一个参数类型一致。(感觉这是个 python bug)

## 参数默认值

- 可以赋默认值：`def test(param1: int, param2: str = '')`
- 注意，带默认值的参数必须放在参数最后

## 返回 tuple

- 函数可以一次 return 多个值，这些值将合为一个 tuple 作为该函数的返回值
- 返回 tuple 时，接收方可以用一个变量接收 tuple 整体，也可以通过多个变量接收 tuple 中的元素
- 如果不需要接收某个元素，则用`_`占位

```Python
def func1():
  return "hello", "world", 1

t = func1()
print(t)  # ("hello", "world", 1)
x, y, _ = func1()
print(x, y) # hello world
```

# 变量

- 用`_`开头的变量可以避免被导出

- Python 有类似 Go 的**并列赋值**语法: `a, b, c = 1, "2", (3)`

## 系统变量

python 有下面几个系统变量，随时可以使用

- `__file__`：当前文件绝对路径
- `__name__`: 当前模块路径
- `__package__`: 当前模块所在的包名
- `__dict__` ：当前类的字典（包括静态属性、动态方法）

# 异常处理

Python 的异常处理与 Java 基本相同。

```python
def test():
  try:
      result = x / y
  except ZeroDivisionError:
      print("division by zero!")
  except Exception as e:
      print(e)
      raise
  else:
      print("result is", result)
  finally:
      print("executing finally clause")
```

- `try`和`except`是必选语句，其它都是可选的
- `except` 的类型要从上到下越来越泛化，最后是`Exception`
- 捕获异常后，就不会向上抛出异常了。如果想处理后继续向上抛出，直接`raise`即可
- 具体类型后可跟`as name`，后面就可以操作这个捕获的异常对象
- `else`是上面都没有满足时走的路径。由于`Exception`是所有异常的父类型，所以通常有`Exception`作为兜底后，就不会走`else`了
- `finally`后是无论如何都会走的代码，不管是否发生异常都会走。如果外层存在循环并且上面处理有**continue**或**break**，仍然会执行`finally`的代码。
  - `finally`可用于关闭文件或连接，这种情况用`with`更合适

# module(模块)

module 是 python 代码重用的的基本单元，一个 module 就是一个`xx.py`文件，这里的`xx`就是模块名。

- 使用`import xx`导入模块名，具体元素需要`xx.a`调用
- 使用`from xx import *`导入模块内的所有全局变量/函数
- 使用`from xx import a, b`精确导入所需元素
- 使用`from ..aa.bb import c`可以导入上层 package 的子 package，这里两个点`..`表示上一层
- 使用`from ...aa.bb import c`可以一直向上找到某个 package，其子 package 包含`aa.bb`

- 使用`from package import item`时，package 可以是**PythonPath**下的系统包/第三方包，也可以是自己当前文件夹下的包；而用`import package`时只能引用 PythonPath 下的包，需要提前把想引用的包的根目录加入到 PythonPath
- `import package as alia_name` 也可以导入自定义包

加入 PythonPath 的两种方法：

1. 临时方法：

```python
import sys
sys.path.append('absolute_path')
```

2. 永久方法 1：
   在对应的 python 安装路径下找到`xxx.pth`文件，在其中添加自定义包所在根文件夹

3. 永久方法 2：
   在`.bashrc`文件中，追加一行：`export PYTHONPATH=${PYTHONPATH}:/absolute_path`

# package(包)

module 要用包来组织，包对应于文件夹，文件夹名就是包名。包和文件夹一样，可以有嵌套结构，通过`.`分隔。

- 使用`import pa.pb.xx` 来导入包
- 在文件夹中放入`__init__.py`表示这个文件夹是一个包，里面的代码会在包第一次被加载时执行
- 在`__init__.py`可以设置变量`__all__ = ["echo", "surround", "reverse"]`如果用`from xxx import *`的导入方式，则会只导入`__all__`指定的元素
- 模块如果被间接导入多次，只会被加载一次，`__init__.py`只会被调用一次
- 如果多个模块都在一个包中，并且这些模块依赖相同的外部包，就可以把这些 import 语句放入`__init__.py`中，避免重复
- 在`__init__.py`可以提前写好`from self.package import class1`来指定可被导出的类或全局变量

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

- 在`.py`中定义的变量是全局变量，可以在函数中使用。但函数中想定义全局变量供外部使用，要用`global xx`声明一下，否则函数中的变量为局部变量。

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

- `del v1`
  删除变量。之后再引用会报"未定义"错误。

# 面向对象

Python 的面向对象支持多继承。

- 多继承时`X(A,B,C)`如果有相同方法名而子类中未指定，则从左向右广度优先遍历。即左边的优先级更高。

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

- 子类访问父类对象的属性时，直接`self.xxx`访问即可
- 子类访问父类对象的函数时，可以用`super().xxx()`

- 子类重写`__init__`函数后就不会自动调父类的构造函数
- 子类调用父类构造函数，可以用`super().__init__(xx,xx)`

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

## `@staticmethod`

与`@classmethod`修饰符相比，函数中无需传入`cls`参数

# 反射

- `getattr(object, name[, default])`
  输入对象、成员名，返回成员的引用

- `hasattr(object，name)`
  判断对象是否有指定名字的成员

- `dir(SampleClass)`
  查看类的成员

- `vars(obj)`
  查看对象实例的成员
