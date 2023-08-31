---
layout: post
title: "Flask snippets"
date: 2023-04-08 22:00:00 +0800
tags: Python Flask
---

记录 Flask 常用片段

```python
# 打印并修改状态码
@app.after_request
def after_request_func(result):
  jsonBody = None
  try:
    jsonBody = request.get_json()
  except Exception as e:
    pass    # no json body
  print('request {} {} {} {} {}, response {} "{}" "{}"'.format(request.remote_addr,request.method, request.path, request.args, jsonBody, result.status_code, result.status, result.data))
  result.status_code = 200
  return result

# 从 url 中取得数组
# http://test.com/test?tag=tag1&tag=tag2
@app.route('/test')
def hello():
    tags = requests.args.getlist('tag')
    print('tags')
```

- `request.headers.get('xxx')`
  获取 header 时，由于 header 的 key 不区分大小写，所以无论大小写都可以获取到正确的结果。 header 在传递过程中 key 的大小写可能被修改。

## 在 request 处理过程中传递全局变量

```python
from flask import (g)

@app.before_request
def func1():
  g.field1 = "test"  # 先定义变量，这样后面使用时不会报未定义

# 后面可以直接读取和写入
g.get("field1", None) # 读取时用 get
```

- 如果没有预先定义，后面使用时会报错：`'_AppCtxGlobals' object has no attribute xxx`

## 用参数渲染页面模板并返回

```html
<head>
    <meta charset="UTF-8">
</head>
<body>
{ % autoescape false % }
  test url {{ url_param }}
{ % endautoescape % }
</body>
</html>
```

- 用`{ { param } }` 设置参数
- 用`{ % xxx % }`设置渲染参数
  - 默认会开启 html 危险符号转义，比如 url 中的`&`符号会被转移为`&amp;`。
  - 用`{ % autoescape false % }`暂时关闭渲染时的转义

```python
from flask import (
    Flask,
    render_template,
    Response,
)

app = Flask(__name__)
app.template_folder = 'path'  # 设置模板文件夹绝对路径
args = {'param1' = 1, 'url_param' = "xxx"}
out_html = render_template('index.html', **args)  # 渲染模板文件夹内的文件
return Response(out_html, mimetype='text/html') # 返回时设置
```
