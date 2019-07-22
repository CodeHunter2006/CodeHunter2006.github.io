---
layout: post
title:  "Go中errgroup包的使用"
date:   2019-07-21 22:00:00 +0800
tags: Go HighConcurrency
---
之前分析过[Go中context包在并发中的使用](/2019/05/21/Go_context/) ，context包提供了基本的并发调用取消功能，但是对于中间发生调用错误的情况无法返回错误结果。而"golang.org/x/sync/errgroup"包在context的基础上，提供了返回错误结果的功能，类似Linux父进程启动多个子进程后，通过`pid_t wait(int *status)`可以返回子进程的退出状态码。

# 核心类定义
``` Golang
// "golang.org/x/sync/errgroup"

type Group struct {
	cancel func()		// 用于cancel的函数对象，与context绑定
	wg sync.WaitGroup	// 用于等待所有子goroutine返回
	errOnce sync.Once	// 保证只有第一个发生的错误被记录，保证线程安全
	err     error		// 记录第一个发生的错误
}
```

# errgroup包中其他函数
```
func (g *Group) Go(f func() error)
```
调用一个子任务函数，一个Group对象可以调用多个子任务函数，然后Wait

```
func (g *Group) Wait() error
```
等待子任务组执行完毕或返回错误，如果返回nil说明正确执行完毕；如果发生错误则返回第一个error

```
func WithContext(ctx context.Context) (*Group, context.Context)
```
通过context.Context对象创建一个绑定的Group对象(nil的Group指针不影响method调用)，以便发生错误后及时调用Context对应的cancel函数。每个子groutine都可以`select <-context.Done()`这样一个任务发生错误，其它任务可以及时取消。

# 使用示例
在源码及对应的doc中有三个很好的示例，这里就不粘贴了，记录一下使用要点：
* 通过`WithContext`创建一个可及时取消的Group，如果发生错误后不需要取消其它子任务，则直接创建一个nil Group即可。
* 调用`group.Go`启动多个任务函数，如果有context要在任务函数中`select <-context.Done()`以便及时退出
* 调用`group.Wait`等待任务完成，或者获得第一个发生的错误

