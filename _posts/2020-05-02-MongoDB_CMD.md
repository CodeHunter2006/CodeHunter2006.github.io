---
layout: post
title: "MongoDB 常用命令"
date: 2020-05-02 23:00:00 +0800
tags: MongoDB
---

## db.createUser()

创建用户

```
db.createUser(
{
    user: "name",
    pwd: "password",
    roles: [
        {
            role: "root/readWrite/read",
            db: "dbname"
        },
        ...
    ]
}
)
```

- 管理员本地执行`mongo`登录进入 mongo 后一定要先执行`use admin`，以 admin 作为验证库，再执行创建用户的命令，否则命令将被执行于 test 库，无法起作用。

## db.auth()

`db.auth('userName', 'dbName')`
检查用户是否有权限操作某库，如果有则返回 1，否则返回 0

## sh.status()

在 mongos 查看集群分片信息

# 其他命令

`db.system.users.find().pretty()`
查询已存在的用户
