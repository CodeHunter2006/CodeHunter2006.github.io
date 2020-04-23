---
layout: post
title: "MySQL Basic"
date: 2020-04-23 23:00:00 +0800
tags: MySQL
---

## PROCEDURE(存储过程)/FUNCTION(存储函数)

两者都是指提前在 MySQL 服务端编译好可被调用的函数，用来提高查询计算速度。
以前之后 FUNCTION 可以有返回值，后来的版本 PROCEDURE 和 FUNCTION 都可以通过 OUT 参数返回值了，两者差别就不大了。

- 一般只有一个返回值的，用 FUNCTION，否则用 PROCEDURE
- 通常不会用到，除非因为网络开销太大，才会把逻辑做到 MySQL 去做，语法只是普通脚本，并不是很好用
