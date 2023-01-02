---
layout: post
title: "Python 语法"
date: 2023-01-07 22:00:00 +0800
tags: Python
---

记录 Python 特殊语法点

## 变量

- 用`_`开头的变量可以避免被导出

## module(模块)

module 是 python 代码重用的的基本单元，一个 module 就是一个`xx.py`文件，这里的`xx`就是模块名。

- 使用`import xx`导入包名，具体元素需要`xx.a`调用
- 使用`from xx import *`导入模块内的所有全局变量/函数
- 使用`from xx import a, b`精确导入所需元素

## package(包)

module 要用包来组织，包对应于文件夹，文件夹名就是包名。包和文件夹一样，可以有嵌套结构，通过`.`分隔。

- 使用`import pa.pb.xx` 来导入包
- 在文件夹中放入`__init__.py`表示这个文件夹是一个包，里面的代码会在包第一次被加载时执行
- 在`__init__.py`可以设置变量`__all__ = ["echo", "surround", "reverse"]`如果用`from xxx import *`的导入方式，则会只导入`__all__`指定的元素

## bytes 与 str

str 是字符串，通常用`""`或`''`构造，底层编码为 Unicode 字符(0-65535)
bytes 是二进制字节数组，用`b''`构造，底层存储字节(0-255)

- str 转 bytes: `b = s.encode('utf-8')`
- bytes 转 str: `s = b.decode('utf-8')`
