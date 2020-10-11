---
layout: post
title: "Golang CAS"
date: 2020-10-11 21:00:00 +0800
tags: Go HighConcurrency
---

![CAS](/assets/images/2020-10-11-Golang_CAS_1.jpg)
**CAS**(Compare And Swap)是 CPU 提供的最基本的原子操作指令，用来实现 Mutex、自旋锁等基础语言组件。本文介绍 Go 中 CAS 的基本使用方法。

# 使用示例

`sync.atomic`包中的函数声明：`func CompareAndSwapInt32(addr *int32, old, new int32) (swapped bool)`

```Go
// 实现多个携程同时累积增 1 的功能
func main() {
  var wg sync.WaitGroup
  var gInt int32

  for i := 0; i < 100; i++ {
    go func(){
      for {
          temp = atomic.LoadInt32(&gInt)
          if ret := atomic.CompareAndSwapInt64(&gInt, temp, temp + 1); ret {
            log.Println("gint from ", temp, "to", temp + 1)
            break;
          }
      }

	    wg.Done()
    }()
  }

  wg.Wait()
  log.Println("last gint:", gInt)
}
```

- 上面"增 1"的动作如果用`gInt++`是无法实现的，因为在底层被编译为三行代码：`tmp := gInt; tmp = tmp + 1; gInt = tmp;`，并发下会发生最终总数偏小的情况。

# CAS 特性

- CAS 四个要素：地址(用于指定变量)、旧值(用于计算)、期望值(用于检查)、新值(检查成功后写入)
- CAS 是由 CPU 底层指令实现的，所以效率极高，在 Intel x86 平台对应`cmpxchg`指令
- 利用 CAS 可以实现乐观锁(自旋重试)，比用 Mutex 在大并发下效率更高
- 高并发下，自旋会消耗 CPU，所以实现 Mutex 时，如果多次自旋未果则进入阻塞状态
- CAS 只支持一个变量的操作，不支持多个变量的操作，如果需要操作多个变量可以用 sync.Mutex

- 多核 CPU 下`cmpxchg`指令如何保证线程安全？
  系统底层进行 CAS 操作时，会检查当前系统是否为多核，如果是多核，则给"总线"加锁，只有一个线程可以加锁成功，再执行 CAS 操作。

- ABA 问题
  利用 CAS 做乐观锁可能发生 ABA 问题，这时需要增加一个版本号，在 CAS 时只有版本号检查通过才算成功
  - 乐观锁实现要点：对象指针、版本号形成一个 pair，对 pair 进行原子操作
  - 也利用 unsafe.Pointer 避免 ABA 问题`func CompareAndSwapPointer(addr *unsafe.Pointer, old, new unsafe.Pointer) (swapped bool)`
  - 如果只是对某个变量 +-1，这种 ABA 其实没什么问题
  - 也可以用 Mutex 解决 ABA 问题
