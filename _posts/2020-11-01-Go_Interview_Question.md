---
layout: post
title: "Go 综合面试题"
date: 2020-11-01 22:00:00 +0800
tags: Go
---

记录一些考点综合的面试题

## 1

下面的代码会输出什么，并说明原因

```Go
func main() {
  runtime.GOMAXPROCS(1)
  wg := sync.WaitGroup{}
  wg.Add(20)

  for i := 0; i < 10; i++ {
    // 下面这种方式，可以定义同名新变量，避免闭包引用变量问题
    // i := i
    go func() {
      fmt.Println("A: ", i)
      wg.Done()
    }()
  }
  for i:= 0; i < 10; i++ {
    gofunc(i int) {
      fmt.Println("B: ", i)
      wg.Done()
    }(i)
  }
  wg.Wait()
}
```

- go 执行的随机性
- 闭包中外部变量引用
- 可以用重定义变量的方式，避免闭包导致同一变量引用

## 2

下面的代码会输出什么，并说明原因

```Go
const s = "Go101.org"

var a byte = 1 << len(s) / 128
var b byte = 1 << len(s[:]) / 128

func main() {
	println(len(s), len(s[:]))
	println(a, b)
}
```

- 未确定类型常量
- 常量在编译期的运算过程
- 移位溢出过程
