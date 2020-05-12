---
layout: post
title: "MongoDB 常用命令"
date: 2020-05-02 23:00:00 +0800
tags: MongoDB
---

# 命令分类

- db(database) 数据对象相关
- rs(replica set) 复制集相关
- sh(sharding cluster) 分片集群相关

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

## rs

`rs.conf()`
查看复制集各成员状态

`rs.status()`
查看复制集信息

`rs.printSlaveReplicationInfo()`
查看复制过程的延迟信息

## sh

`sh.status()`
在 mongos 查看集群分片信息

# 其他命令

`db.system.users.find().pretty()`
查询已存在的用户
