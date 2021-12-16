---
layout: post
title: "Go Representable/Assignable/Comparable"
date: 2020-11-07 22:00:00 +0800
tags: Go
---

数据的类型判定严格程度：Representable(可表示，常量) < Assignable(可赋值) < Comparable(可等于比较) < Comparable(可顺序比较)

# representable

如果符合下面三项，我们说"一个常量 x 可以被表示为某种类型 T 的变量"：

- x 在 T 所定义的集合范围内
- 如果 T 是一种浮点型，x 可以近似到 T 的精度而不溢出
- 如果 T 是 complex(复数)类型，x 的实部和虚部分别可以表示为 T 的实部和虚部

# assignable

符合下面情况，我们说"一个变量 x 可以被赋值给 T 类型的变量"

- x 本身就是 T 的类型的变量
- x 的类型 V 和 T 是潜在的相同类型，并且至少 T 和 V 中有一个是没有被`type`关键字明确定义的
- T 是 interface 类型，并且 x 实现了 T
- x 是一个双向 channel，T 是某种(双向/只读/只写)channel
- x 是 nil 值，而 T 是下面这几种类型：pointer, function, slice, map, channel, interface
- x 是编译期未确定类型的常量，可被表示为 T 类型

# comparable

满足 assignable 的基础上才考虑 comparable，comparable 的最低要求是可以等于比较`==`和`!=`，然后才考虑顺序比较`<`、`<=`、`>`、`>=`

- Boolean 可等于比较
- Integer 可顺序比较
- Floating-point 可顺序比较
- Complex 可等于比较
- String 可顺序比较。从第一个索引开始比较每个字符，如果前面的字符都相同则长度长的更大
- Pointer 可等于比较。如果指针指向同一对象地址或都是 nil 值，则相等
- Channel 可等于比较。如果两个 channel 变量的值都是同一个 make 调用创建的，或者都是 nil 则相等
- Interface 与 Complex 类似，可等于比较。Interface 的 Type 和 Value 两个指针值都相同时才相等
  - 推论：值类型的 Receiver 对应的 interface 变量，永远无法相等，因为 Value 指向不同地址
  - 如果两个 interface 变量进行比较，而其动态类型不匹配，则会引发 panic
- `interface{}`是一种特殊的 Interface，可以根据实际类型决定比较指针还是指针指向的对象
- Struct 可等于比较。所有字段都相等时相等
- Array 可等于比较。长度、类型相同，所有元素都相等时相等

- Slice、map、function 不可比较，除非是和 nil 值比较。

- 可以利用 function 的"不可比较"特性，让指定的 struct 不能被比较：

  ```Go
  // DoNotCompare can be embedded in a struct to prevent comparability.
  type DoNotCompare [0]func() // 数组 len 为 0，不占空间
  type T struct {
    name string
    DoNotCompare
  }
  func main() {
    fmt.Println(T{} == T{}) // 由于存在 func 类型对象比较，编译不过
  }
  ```
