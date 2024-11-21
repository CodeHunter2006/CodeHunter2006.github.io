---
layout: post
title: "Go 泛型"
date: 2022-08-13 10:11:00 +0800
tags: Go
---

![Generic](/assets/images/2022-08-13-Go_generics_1.png)
记录 Go 泛型基本语法。

# 示例

```Go
type signed interface {
	~int | ~int8 | ~int16 | ~int32 | ~int64	// Type Approximation
}

func isEqual[T comparable](a, b T) bool {
	return a == b
}

type gTest[T signed, S ~string] struct {
	A, B T
	Strs S
}

func testGeneric() {
	v1 := gTest[int32, string]{A: 1, B: 2, Strs: "3"}
	v2 := gTest[int32, string]{A: 1, B: 1, Strs: "2"}
	fmt.Println(isEqual[int32](v1.A, v1.B)) // actually, no need for type declaration
	fmt.Println(isEqual(v2.A, v2.B)) // auto detect type
}
```

# 语法

- 类似 C++ 的`<>`，Go 中以`[]`作为泛型类型声明符号
- 函数声明方式`func XXX[T any](param T) (T, error) {...}`
  - `T` 是类型参数
  - `any` 这个位置是**约束集**，可以用多种方式约束
  - 可以声明多种类型，以`,`隔开
- 约束集可选方式：
  - `xxxxer` 某种 interface
  - `int32` 某种类型
  - `~int32` 底层以`int32`实现的各种类型
  - `int32 | int64 | float32` 用`|`间隔各种可能类型
  - `any` 新增的关键字，"任意类型"，其本质是`interface{}`的一个别名
    - **应该在所有地方尽可能用 any 替代 interface{}，更易理解**
  - `comparable` 新增的关键字，"可比较"，要求该类型需满足`==`和`!=`
  - 另外，Go 定义了一个常用的约束包: https://pkg.go.dev/golang.org/x/exp/constraints
- 结构体声明方式与函数类似`type XXX[T any] struct{...}`
- interface 语法也进行了扩展，可以增加**类型逼近**(Type Approximation)声明
- 通常调用泛型函数/结构体之前要用`[]`声明实际类型，但有些情况下可以类似 C++ 的 auto，自动根据上下文推断类型，从而可以**省略类型声明**

# 限制

- 泛型的目的

  - 利用静态语言的特点安全的传递参数，做编译时检查
  - 对类型进行抽象，弥补 interface 对抽象的不足

- 缺陷
  - Go 的泛型底层是用 interface 实现的，所以性能相对 C++ 会较差
  - 如果为了实现静态编译的性能，可以用第三方代码生成工具的泛型实现方式
  - 某些情况下泛型和接口是通用的，这种情况下应优先使用接口，因为接口设计更成熟、简单，泛型在抽象和性能都没有改进

# 其他

## 返回零值

下面编译会报错 "cannot use nil as T value in return statement"

```Go
func test[T any]() (T, error) {
  return nil, nil
}
```

可以用两种办法解决：

```Go
func test[T any]() (T, error) {
  var v T // 用零值局部变量返回
  return v, nil
}

// 利用具名返回值更优雅
func test[T any]() (t T, err error) {
  return
}
```

## 自带的泛型函数

- min/max
  - 从 1.21 版引入的泛型函数，可以方便计算
