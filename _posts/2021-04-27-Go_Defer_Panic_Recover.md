---
layout: post
title: "Go defer 原理及常见问题"
date: 2021-04-27 23:00:00 +0800
tags: Go
---

![go select](/assets/images/2021-04-27-Go_Defer_1.png)
Go 的**defer**关键字实现了栈式"善后操作"，类似 java 的 finally，极大的简化了开发复杂度，这里记录其基本原理及常见问题。

# 基本结构

defer 的基本结构如下：

```Go
type _defer struct {
    siz     int32       // 参数大小
    started bool        // defer是否被调用过的标识
    sp      uintptr     // 函数栈指针
    pc      uintptr     // 程序计数器
    fn      *funcval    // 函数地址
    _panic  *_panic     // 指向panic链表
    link    *_defer     // 指向自身结构的指针，用于链接多个defer
}
```

- link 使得 defer 可以形成**链式结构**，如上图每次添加 defer 结点都添加到单链表头部，所以执行时类似栈(FILO)
- sp pc fn 使得 defer 真正执行时可以调用函数
- **根 defer 指针在栈上**，在 return 或 panic 后会被执行。也由于根 defer 指针在 goroutine 栈上，所以 defer 只对当前 gouroutine 起作用
- **panic**链表的根结点在 defer 结点中，所以必须在 defer 中才能处理 panic，比如进行 recover

- 新增 defer 结点时，其函数参数是一个表达式的值，也就是当时**立刻执行了表达式**而不是后来执行的
- defer 结合**闭包**时，闭包中的变量是以引用的方式保存的(变量地址)，所以在 defer 执行时可以读取到最新值并可以修改
- **注意**一般的 defer 结点会被创建在**栈**上以保证高效率，但是如果 defer 被放在 for 循环中，则会被编译器检测到，将结点放到**堆**上，可能造成性能问题

![go select](/assets/images/2021-04-27-Go_Defer_2.png)

panic 的基本结构如下：

```Go
type _panic struct {
	argp      unsafe.Pointer    // 一个指针，指向defer调用的参数的指针
	arg       interface{}       // panic传入的参数
	link      *_panic           // 是一个链表结构，指向上一个调用的_panic
	recovered bool              // 是否已被recover 恢复
	aborted   bool              // panic是否被强行中止
}
```

- 新的 panic 会被挂在当前 goroutinue 的 panic 链的最前面
- 一旦开启了 panic 那么会 for 循环执行当前 goroutine 的 defer 链
- 如果一直没有 recover，那么 panic 会一直向最上层调用栈传递，直到最后执行 fatalpanic 终止程序
- 在执行 recover 后会将 panic 的 recovered 标记为 true，之后会继续执行完所有当前 goroutine 的 defer，然后返回上层调用栈，之后正常继续执行

# 常见用法

- 打开资源时，如果判断没有 error，则立即`defer xxx.Close()`以确保资源释放。类似的还有 Lock 等。
- 对于 return 后才执行的代码，由于 return 的位置都添加太麻烦，可以利用 defer
- 对于发生了 panic 的情况，可以在 defer 中捕获并处理
- defer 是栈式结构，可以利用这个特点实现逻辑

- 示例：

[实现 unlock](/2020/11/19/Go_Defer_Unlock/)
[关闭 time.Timer](/2021/03/31/Go_select/#timerafter-泄漏问题)
[封装 panic 为 error](https://codehunter2006.github.io/2020/03/22/Go_error_best_practice/#%E5%BC%82%E5%B8%B8%E5%A4%84%E7%90%86%E7%9A%84%E4%B8%80%E4%BA%9B%E8%AE%BE%E8%AE%A1%E7%82%B9)
[及时关闭 channel](https://codehunter2006.github.io/2020/10/25/Go_Concurrency_Patterns/#explicit-cancellation)

# 其他

- 最初 defer 的结点都是在堆上，所以性能较差，高性能的代码不敢用 defer。后来**Go 1.13**进行了优化，使得性能完全没有问题了。
  但是 for 循环中的 defer 会导致结点又分配在堆上，并且数量大时性能会更差
  - Go 编译器会检查 for 循环的循环次数，如果超过 1，则分配到堆上
  - 我们要避免在 for 循环里执行 defer，同时注意`goto`语句可能形成类似的循环结构
- **Go1.14**对 defer 有了进一步优化，编译器直接把 defer 函数插入到函数末尾执行(Open-coded 技术)，
  避免了运行时的 deferproc 及 deferprocStack 操作，但要求 defer 数量小于 8，否则取消优化
