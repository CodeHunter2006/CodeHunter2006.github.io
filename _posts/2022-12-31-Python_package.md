---
layout: post
title: "Python Package"
date: 2022-12-31 22:00:00 +0800
tags: Python
---

Python 常用包记录

# python 自带包

## 内置函数

- `type(object)`
  返回对象的类型

- `isinstance(object,classinfo)`
  判断对象是否匹配类型，考虑继承关系
  - `classinfo`可以为元组形式`isinstance(object,(str,int,list))`，表示只要满足其中一个类型即可

```python
class A:
    pass

class B(A):
    pass

isinstance(A(), A)    # returns True
type(A()) == A        # returns True
isinstance(B(), A)    # returns True
type(B()) == A        # returns False
```

## string

### join

- `string.join(iterable)`
  将当前字符串作为分割符，将迭代列表元素进行分割后，组成新的字符串

```python
print(' '.join(['Python', 'is', 'a', 'fun', 'programming', 'language']))
# Output: Python is a fun programming language
```

### format

```python
# 打印 hello world!
"{} {}!".format("hello", "world") # 直接按顺序占位
"{1} {0}!".format("world", "hello") # 通过下标占位
"{name1} {name2}".format(name1="hello", name2="world")  # 通过参数名占位
"{price:.2f} dollars!".format(price=2)  # 2.00 dollars! 参数名占位加指定格式

# 用 f/F string 可以实现用当前变量作为参数名占位格式化
name1 = "hello"; name2 = "world"
f"{name1} {name2}!"
```

- [特殊格式示例](https://docs.python.org/3/library/string.html#formatexamples)

### strip

- `str.strip([chars])`

删除字符串前后的指定字符，如果不传参数默认为空格和换行。
传入的字符串会被看作是字符数组，每个字符都独立被用于删除操作。

## datetime

datetime 包可以用于时间的各种计算，其中有几个关键类型：

- `date`年月日等日期相关
- `time`时分秒等时间相关
- `datetime`年月日时分秒完整信息
- `timedelta`两个`date/time/datetime`之间的差
- `timezone`时区信息

```python
from datetime import datetime, timedelta, timezone

time1 = datetime.now(tz = timezone.utc)   # 创建 datetime，不传时区参数则默认系统时区
time1 += timedelta(days = 5)    # 可以调整时间
timeStr = tarTime.strftime("%Y-%m-%d %H:%M:%S")  # datetime 转字符串
print(timeStr)  # 2022-01-05 09:19:52

# 字符串转 datetime，同时设定时区
time2 = datetime.strptime(timeStr, "%Y-%m-%d %H:%M:%S").replace(tzinfo = timezone.utc)
time2 -= timedelta(minutes = 5)
print(time2)    # 2022-01-05 09:19:52

during = time1 - time2  # 计算差值，获得 timedelta 类型
print(during.total_seconds())
```

## base64

base64 格式用 64 个可打印字符来表示二进制数据，操作过程输入输出都是 bytes 类型。

```python
import base64

base64_data = base64.b64encode(b'abc')
print(base64_data.decode())
origin_data = base64.b64decode(base64_data)
print(origin_data.decode())
```

## json

json 操作

- `str1 = json.dumps(obj)` 对象转 json 字符串
- `obj1 = json.loads(str1)` json 字符串转对象

## sys

系统相关

- `sys.path.insert(0, '/tmp')`
  将指定的路径加入到 path 中的最前面(第 0 个位置)

## traceback

用于打印调用栈

- `traceback.print_stack()`
  打印调用栈

## threading

多线程相关

```python
import threading

lock = threading.Lock() # 创建一个互斥锁

lock.acquire()  # 获取锁

lock.release()  # 释放锁
```

# 第三方包

## pyjwt

实现 jwt(JSON Web Token) 编解码

```python
import jwt

body = {'a': 'valueA'}  # 要保存的 kv 对象
secret = "123456"       # 秘钥
tokenStr = jwt.encode(body, secret, algorithm='HS256')
print("token is: " + tokenStr)  # 加密后的乱码

dictResult = jwt.decode(tokenStr, secret, algorithms=['HS256'])
print(dictResult.get('a'))  # valueA
```

## pyyaml

操作 yaml 文件

```python
import yaml

yamlContent = 'key1: "value1"'
yamlDict = yaml.load(yamlContent)
```

## pipdeptree

可以用命令查看 python 包的依赖关系

- `pipdeptree`
  显示依赖树

## requests

简化 http 请求

```python
import requests

response = requests.get("https://www.baidu.com")
print(response.text)	# 返回的文本响应

bodyStr = "{\"content\": \"123\"}"
auth=('username', 'password')		# basic auth
headers={"Content-Type":"application/json"}
response = requests.post("https://xxx/xxx", data = bodyStr, headers = headers, auth = auth)

print(response.status_code)
responseDict = response.json()  # 将结果 json 转为级联 dict
```

## pprint

打印嵌套对象。
pprint 只能用于一般测试，因为它无法把输出结果放到字符串中，在有 logger 时会使用不便，不过 logger 一般都有嵌套打印功能。

```python
from pprint import pprint

x = {'a': {'b': 'c'}}
pprint(x) # {'a': {'b': 'c'}}
```

## pyflakes

提供基本的语法扫描功能

- `python -m file1.py file2.py ...`
  扫描指定代码 path

- `python -m pyflakes ./folder1/* | grep -Ev 'but never used|imported but unused|redefinition of unused|may be undefined|unable to detect undefined names'`
  扫描指定文件夹下所有`.py`文件，并且忽略常见的 warning
