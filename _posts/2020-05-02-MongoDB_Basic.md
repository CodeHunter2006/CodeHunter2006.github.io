---
layout: post
title: "MongoDB 基础"
date: 2020-05-02 23:00:00 +0800
tags: MongoDB
---

![MongoDB](/assets/images/2020-05-02-MongoDB_Basic_1.jpeg)

- mongodb 属于 NoSQL(Not only SQL)，提供的查询语句更加人性化
- 与 mysql 可以类比，库(database)、表(table collection)、行(row document)。其中 mongodb 的行是一个 json 字符串，其中包含了每列名称和值。
- 库、表、列名，都不需要提前创建，随插入数据时会自动创建
- 插入一行中的字段名(列名)不需要提前声明，每一行列可以不同
- 插入一行数据时，会自动填写"\_id"，底层会按照聚集索引存储
- 的命令行操作非常人性化，符合面向对象风格，可以在对象名后面调用函数完成操作。
- 遇到不会的命令时，可以调用 help 函数，也可以按两下 Tab 进行自动提示
- mongodb 的查询语句相比 sql 要简单直接，只能对单表操作，不存在联表查询优化，多表间的关系要调用者自己去处理
- 验证库，远程登录时要指定验证库。创建用户时，要指定该用户管理的库。管理员权限角色(root)，要用 admin 作为验证库。
- mongodb 只能针对库设定权限，不能针对表
