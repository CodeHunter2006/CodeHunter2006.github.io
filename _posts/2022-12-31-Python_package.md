---
layout: post
title: "Python Package"
date: 2022-12-31 22:00:00 +0800
tags: Python
---

Python 常用包记录

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
