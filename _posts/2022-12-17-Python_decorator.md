---
layout: post
title: "Python Decorator"
date: 2022-12-17 22:00:00 +0800
tags: Python
---

![Decorator](/assets/images/2022-12-17-Python_decorator_1.jpeg)
工作中用到了 Decorator，正好看到一篇[比较好的文章](https://www.datacamp.com/tutorial/decorators-python)，翻译作为笔记。

**装饰器**是一种常用的设计模式，允许你为已存在的对象添加新功能而不改变对象的内部结构。

- Python 的装饰器和 Hook 函数和"面向切面编程"起相同的效果

首先要重点说明的是在 Python 中**函数是一等公民**。这意味着它们可以支持如下操作：被作为参数传递、作为一个函数的返回值返回、修改、设置给一个函数变量。这是接下来理解装饰器模式的基础概念。

## 将函数赋值给一个变量

```Python
def plus_one(number):
    return number + 1

add_one = plus_one  # 这里将函数赋值给变量

add_one(5)          # 在这里执行函数
# 6
```

## 在函数内定义另一个函数

```Python
def plus_one(number):
    def add_one(number):
        return number + 1

    result = add_one(number)  # 执行内部定义的函数
    return result

plus_one(4)
# 5
```

## 把函数作为其他函数的参数传递

```Python
def plus_one(number):
    return number + 1

def function_call(function):
    number_to_add = 5
    return function(number_to_add)  # 执行传入的函数，并将函数执行结果作为本函数结果返回

function_call(plus_one)
# 6
```

## 将函数内部定义的函数作为返回值返回(生产函数的函数)

```Python
def hello_function():
    def say_hi():
        return "Hi"
    return say_hi

hello = hello_function()
hello()
# 'Hi'
```

## 嵌套函数中，内部函数可以共享外部函数的变量(这种模式也叫做"闭包")

```Python
def print_message(message):
    "Enclosong Function"
    def message_sender():
        "Nested Function"
        print(message)

    message_sender()

print_message("Some random message")
# 'Some random message'
```

## 创建装饰器

有了前面的准备，终于可以实现一个装饰器了。我们的装饰器只实现一个简单地功能：将被装饰的函数的返回的字符串改写为大写。
为了实现这个功能，我们需要利用到前面说的**闭包**结构：

```Python
def uppercase_decorator(function):  # 这里传入被装饰的函数
    def wrapper():
        result = function()             # 在内部函数执行被装饰的函数，结果是一个字符串
        make_uppercase = result.upper() # 将结果字符串转为大写
        return make_uppercase

    return wrapper # 将闭包函数返回
```

接下来定义一个返回字符串的函数，用上面定义好的装饰器函数进行装饰：

```Python
def say_hi():
    return 'hello there'

decorate = uppercase_decorator(say_hi)
decorate()
# 'HELLO THERE'
```

然而，Python 提供了更方便的方法进行装饰。我们只要简单的在装饰器函数前加上`@`并放在被装饰函数上面即可：

```Python
@uppercase_decorator
def say_hi():
    return 'hello there'

say_hi()
# 'HELLO THERE'
```

## 在被装饰函数上使用多个装饰器

基于前面的代码，我们再定义一个新的装饰器：将被装饰函数返回的字符串用空格分割为字符串数组。

```Python
def split_string(function):
    def wrapper():
        result = function()
        splitted_string = result.split()
        return splitted_string

    return wrapper
```

```Python
@split_string
@uppercase_decorator
def say_hi():
    return 'hello there'

say_hi()
# ['HELLO', 'THERE']
```

你可能注意到了，上面两个装饰器的执行流程只能是**从下到上**的，即先转换为大写再分割为数组。
为了方便记忆，你可以按照**先执行内部(原始函数、第一层装饰器)，再执行外部(更外层装饰器)**的顺序理解。
同一个装饰器也可以被装饰在一个被装饰函数上多次，每一次装饰都是独立的。这也是符合**装饰器模式**的。

## 在装饰器中截取被装饰函数的参数

装饰器被调用时，传给被装饰函数的参数会先传给内部的 wrapper 函数：

```Python
def decorator_with_arguments(function):
    def wrapper_accepting_arguments(arg1, arg2):
        print("My arguments are: {0}, {1}".format(arg1,arg2)) # 截获并打印参数
        function(arg1, arg2)  # 调用被装饰函数时传入截获的参数，中间可以做修改
    return wrapper_accepting_arguments

@decorator_with_arguments
def cities(city_one, city_two):
    print("Cities I love are {0} and {1}".format(city_one, city_two))

cities("Nairobi", "Accra")
# My arguments are: Nairobi, Accra
# Cities I love are Nairobi and Accra
```

## 在装饰器中截取更通用的参数

定义内部 wrapper 函数时，可以用`(*args, **kwargs)`截取通用参数。
其中`*args`是顺序传入参数；`**kwargs`是通过 key-value 方式指定的参数。

```Python
def a_decorator_passing_arbitrary_arguments(function_to_decorate):
    def a_wrapper_accepting_arbitrary_arguments(*args,**kwargs):
        print('The positional arguments are', args) # 本质是一个数组
        print('The keyword arguments are', kwargs)  # 本质是一个dict
        function_to_decorate(*args)
    return a_wrapper_accepting_arbitrary_arguments

@a_decorator_passing_arbitrary_arguments
def function_with_arguments(a, b, c):
    print(a, b, c)

function_with_arguments(1, 2, 3)
# The positional arguments are (1, 2, 3)
# The keyword arguments are {}
# 1 2 3

function_with_arguments(c=1, a=2, b=3)
# The positional arguments are ()
# The keyword arguments are {'c':1, 'a':2, 'b'=3}
# 2 3 1
```

## 带有参数的装饰器

有时需要给装饰器增加参数，实现方案同样基于"闭包"。装饰器本质上就是一个函数，这个函数执行时会返回一个可执行的函数。
为了添加装饰器参数，我们只需要在外层再套一层函数添加参数，这层的参数将被作为闭包变量使用。

```Python
def decorator_maker_with_arguments(decorator_arg1, decorator_arg2, decorator_arg3):
    def decorator(func):
        def wrapper(function_arg1, function_arg2, function_arg3) :
            "This is the wrapper function"
            print("The wrapper can access all the variables\n"
                  "\t- from the decorator maker: {0} {1} {2}\n"
                  "\t- from the function call: {3} {4} {5}\n"
                  "and pass them to the decorated function"
                  .format(decorator_arg1, decorator_arg2,decorator_arg3,
                          function_arg1, function_arg2,function_arg3))
            return func(function_arg1, function_arg2,function_arg3)

        return wrapper

    return decorator

pandas = "Pandas"
@decorator_maker_with_arguments(pandas, "Numpy","Scikit-learn")
def decorated_function_with_arguments(function_arg1, function_arg2,function_arg3):
    print("This is the decorated function and it only knows about its arguments: {0}"
           " {1}" " {2}".format(function_arg1, function_arg2,function_arg3))

decorated_function_with_arguments(pandas, "Science", "Tools")
# The wrapper can access all the variables
#     - from the decorator maker: Pandas Numpy Scikit-learn
#     - from the function call: Pandas Science Tools
# and pass them to the decorated function
# This is the decorated function, and it only knows about its arguments: Pandas Science Tools
```

## 调试装饰器

你应该已经注意到了，装饰器会把原函数封装起来。这导致原函数的名称、文档字符串和参数列表被封装在闭包中。例如当你想获取`decorated_function_with_arguments`的**元数据**时，我们只能看到 warapper 的闭包元数据。这将导致难以调试。

```Python
decorated_function_with_arguments.__name__
# 'wrapper'
decorated_function_with_arguments.__doc__
# 'This is the wrapper function'
```

为了解决这个问题，Python 提供了一个`functools.wraps`装饰器。这个装饰器可以将原函数的元数据拷贝到闭包中。示例如下：

```Python
import functools

def uppercase_decorator(func):
    @functools.wraps(func)
    def wrapper():
        return func().upper()
    return wrapper

@uppercase_decorator
def say_hi():
    "This will say hi"
    return 'hello there'

say_hi()
# 'HELLO THERE'

say_hi.__name__
# 'say_hi'
say_hi.__doc__
# 'This will say hi'
```

建议你尽量在所有定义装饰器的地方使用`functools.wraps`，这将节省你大量的令人头疼的调试时间。

## Python 装饰器总结

装饰器可以动态的改变原函数/方法/类而不必使用继承或修改原函数。使用装饰器可以确保你的代码**DRY**(Don't Repeat Yourself)。

装饰器的常用场景：

- 在 Python 框架中进行授权处理
- 打 log
- 度量函数执行时间
- 同步处理

想要了解更多装饰器知识，可以查看 Python 官方文档[Decorator Library](https://wiki.python.org/moin/PythonDecoratorLibrary)
