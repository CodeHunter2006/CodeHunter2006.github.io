---
layout: post
title: "Go Package Notes"
date: 2019-12-28 22:00:00 +0800
tags: Go
---

Go 常用包的功能，及注意点

## flag

用于获取命令行参数，同时可以设置默认值

可以获得各种类型的参数，无法获得无参数标记的参数

## os.signal

可以设置监听哪些系统信号

## rand

使用前要先做一个种子，否则其实不随机。
rand.Seed(time.Now().Unix())

## pool

提供对象池接口，用于缓存对象，避免频繁申请、释放内存造成的性能问题。

- 池不可以指定大小，大小受限于 GC 的临界值
- 对象的最大缓存周期是 GC 周期，当 GC 调用时没有引用的对象会被清理掉
  - pool 包在 init 中注册一个 poolCleanup 函数用于清除所 pool 里的缓存对象，每次 GC 之前会被调用
- Get 方法返回时是返回池中任意一个对象，没有顺序；如果池中没有对象，会调用指定的 New 方法生成一个；如果没有指定 New 方法，那么返回 nil

## fmt

`%+v` 格式符表示将数据结构打印出来
`%[2]v%[2]v%[1]v` 可以通过这种形式选定后边的第几个参数，避免重复传入相同参数

## sync.WaitGroup

可以为多个 goroutine 设置统一起跑时机或统计统一结束时机。

## log

log 相关

`log.Panic()`
输出 panic 级别日志后，开启 panic

`log.Fatal()`
输出 fatal 级别日志后，执行 os.Exit()
