---
layout: post
title: "Go 综合面试题"
date: 2020-11-01 22:00:00 +0800
tags: Go
---

记录一些考点综合的面试题。

(为了避免提示，题目只有序号没有标题)

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

## 3

- 如下两段代码会输出什么？`struct{}`的指针能否比较？为什么？

```Go
// 代码 1
type People struct {}
func main() {
 a := &People{}
 b := &People{}
 fmt.Println(a == b)
}
```

```Go
// 代码 2
type People struct {}
func main() {
 a := &People{}
 b := &People{}
 fmt.Printf("%p %p\n", a, b)
 fmt.Println(a == b)
}
```

- 答案：

  - 代码 1 会输出：`false`
  - 代码 2 会输出：`true`

- 思路：
  - 按照官方说法，`struct{}`(空对象)会统一在系统中占用同一资源，所以节省容量。上面代码 2 是符合的
  - 由于 Go 的逃逸检测，`fmt.Printf(...)`函数使用了反射，所以`a`、`b`两个对象被检测为逃逸，并且指向了同一个全局地址`zerobase`
  - 所有 0 字节容量的堆对象都会指向`zerobase`这个地址，所以相等，是正确的结果
  - 由于 Go 编译器的优化，代码 1 的对象被认为没有逃逸在栈中，两个栈中对象的地址一定不同，所以编译器直接填写了`false`值
    - 在编译时添加`-gcflags="-N -l"`参数，就可以不进行编译优化，代码 1 就会打印`true`了。当然，这种方法是不可取的
- 实际使用：
  - 实际中不会很多，在不关心 struct 成员变量只关心接口函数的情况下才会使用`struct{}`指针
  - 使用中不能依赖"空对象指针是否相等"作为逻辑条件，类似 map 的顺序不能被依赖一样
- 其他：
  - `println`函数不会造成逃逸，但是这个函数官方本身不再支持，也不该这么依赖

## 4

- 下面代码会输出什么？

```Go
func main() {
    var e error
    e = GetErr()
    log.Println(e, e == nil)
}

type MyErr struct {
    Msg string
}
func (m *MyErr) Error() string {
    return "test"
}
func GetErr() *MyErr {
    return nil
}
```

- 错误答案：
  `nil true`

- 真实情况的输出：
  `nil false`

- 为什么会输出`false`?
  `error`是一个 interface，底层可能有两种数据格式：
  - `runtime.eface`表示不包含任何方法的空接口
  - `runtime.iface`表示包含方法的接口

```Go
type eface struct {
  _type *_type
  data  unsafe.Pointer
}
type iface struct {
  tab *itab
  data unsafe.Pointer
}
```

- 可以看到只有`type`和`pointer`相同的情况下，`runtime.eface`才符合`==`。
  由于上面`GetErr()`返回的类型是`*MyErr`，所以被赋值的`e`的类型被设为`eface{*MyErr,nil}`；
  而外面的`nil`interface 的类型是`eface{nilType,nil}`，所以`(e == nil) == false`

- 应对方案：
  - 通常我们函数返回类型时要用 error 这种 interface 类型，这样返回的`nil`就不会带类型，就能匹配`==`
  - 如果返回的类型不是 interface，那调用者要注意和 nil 的比较情况
- 比如下面这种方式就是正确的：

```Go
func GetErr() error {
    return nil
}
```

# 5

- 下面代码会如何运行？输出什么？

```Go
func main() {
  s := []int{1,2,3,4,5}
  for _, v:=range s {
    s =append(s, v)
    fmt.Printf("len(s)=%v\n",len(s))
  }
}
```

- `range`是 Go 的语法糖，可以理解为一个只触发一次的函数，调用时就确定了 len
- sli、map 是相同逻辑

# 6

- 下面代码会输出吗？

```Go
func main() {
  runtime.GOMAXPROCS(1)
  go func() {
    fmt.Println("sub goroutine")
  }()

  for {}
}
```

- 在 1.14 版(包括)之后，可以成功打印"sub goroutine"；在 1.14 版之前会锁死；
