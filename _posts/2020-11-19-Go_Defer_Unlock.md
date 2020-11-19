---
layout: post
title: "Go defer unlock 总结"
date: 2020-11-19 22:00:00 +0800
tags: Go
---

![defer unlock](/assets/images/2020-11-19-Go_Defer_Unlock_1.jpg)
实际工作中经常遇到 Lock 后需要 Unlock 的情况，这里总结一下优化过程。

1. 一般情况
   假设想给某 map 变量加锁后使用

```Go
var mu sync.Mutex
var mVar map[int]int = make(map[int]int)

func func1(){
  mu.Lock()
  defer mu.Unlock()

  for k, v := range mVar {
    doSomeThing(k, v)
  }

  doSomeOthers()
}
```

- 上面的处理缺点是`doSomeOthers()`调用时其实已经不需要 Lock 了，会阻塞其他携程

2. 将 Unlock 提前

```Go
func func1(){
  mu.Lock()
  for k, v := range mVar {
    doSomeThing(k, v)
  }
  mu.Unlock()

  doSomeOthers()
}
```

- `Unlock()`在`doSomeOthers()`前执行，避免阻塞其他携程
- 但是存在隐患：如果`doSomeThing`中发生 panic，并且在外层 recover 了，则会缺少 Unlock 而可能导致死锁

3. 匿名函数 + defer

```Go
func func1(){

  func(){
    mu.Lock()
    defer mu.Unlock()
    for k, v := range mVar {
      doSomeThing(k, v)
    }
  }()

  doSomeOthers()
}
```

- 匿名函数中使用 defer，保证`doSomeThing`发生 panic 也能正确 Unlock
- 匿名函数立即执行，在执行`doSomeOthers`前就会 Unlocks
