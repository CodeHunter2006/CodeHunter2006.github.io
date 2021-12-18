---
layout: post
title: "Gin Middleware"
date: 2021-12-14 22:00:00 +0800
tags: Http DesignPattern
---

![Middleware](/assets/images/2021-12-14-Gin_Middleware_1.jpeg)
Gin 提供了友好的 middleware 接口，方便在 request 处理前后进行通用处理。

引用：
[Gin 中的 Context 包的 Next](https://blog.csdn.net/heart66_A/article/details/100060865)

## 使用示例

```Go
package main

import (
	"fmt"
	"github.com/gin-gonic/gin"
)

func main() {
	router := gin.Default()
	signature := ""

  // 最外层 middleware
	router.Use(func(c *gin.Context) {
    // handler 前
    signature += "A"
    fmt.Println("before:A", signature)

    // 挂起当前函数的执行流程，执行中间的 middleware 和 handler 处理
    c.Next()

    // handler 后
    signature += "A"
    fmt.Println("after:A", signature)
  })

  // 中层 middleware
  router.Use(func(c *gin.Context) {
    signature += "B"
    fmt.Println("before:B", signature)
	})

  // 注册 handler，最内层
	router.GET("/hello", func(c *gin.Context) {
    signature += "C"
		fmt.Println("GET:C", signature)
	})

	router.Run()
}
/*
before:A A
before:B AB
GET:C ABC
after:A ABCA
*/
```

- middleware 可以设置多个，这里以两个举例。可以看出调用`Use`的顺序决定了 middleware 的层次关系，
  就像一个**洋葱**一样，先调用`Use`加入的 middleware 包在洋葱的最外面、洋葱的中心是 handler。
  程序的执行顺序就像用签子从中心扎穿洋葱的过程，preA->preB->hanlderC->postB->postA
- 在程序执行到 middleware 中的流程时，可以用`c.Next()`/`c.Abort()`进行流程控制：

  - `c.Next()`表示"执行完剩下的(middleware+handler)再继续执行本流程"的`post`部分
  - `c.Abort()`表示"执行完本流程后，不再执行剩下的(middleware+handler)逻辑，直接返回最终结果"
    - `c.Abort()`后的本 middleware 的代码会正常执行完毕，不受影响
    - `c.Abort()`只有在`c.Next()`之前才起作用，其本质是通过控制责任链的 index 跳过所有后续节点

- 另外，可以用`c.IsAborted()`查询当前`Abort`状态

## 底层实现

其实 middleware 的底层是 Chain of Responsibility Pattern，即把一系列 middleware 函数放在一个 hook 数组里逐个执行，
利用数组的 index 控制流程。

```Go
func (c *Context) Next() {
	c.index++
	for c.index < int8(len(c.handlers)) {
		c.handlers[c.index](c)
		c.index++
	}
}

const abortIndex int8 = math.MaxInt8 / 2
func (c *Context) Abort() {
	c.index = abortIndex
}
```
