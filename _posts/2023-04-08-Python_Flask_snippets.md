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
  print('request {} {} {} {}'.format(request.method, request.path, request.args, jsonBody))
  result.status_code = 200
  return result
```
