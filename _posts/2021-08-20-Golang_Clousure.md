---
layout: post
title: "Golang Clousure"
date: 2021-08-20 22:00:00 +0800
tags: Go
---

![Clousure](/assets/images/2021-08-20-Golang_Clousure_1.jpeg)
**闭包(Clousure)**是 Javascript 最常用的函数封装调用方式，在早期 JS 版本中甚至是实现面向对象的唯一方式。
在 Go 中也会经常用到，这里总结一下 Go 中闭包实现原理。

# 定义

闭包是由**函数**及其**相关引用环境**组合而成的实体

示例：

```Go
func f(i int) func() int {
	return func() int {
		i++
		return i
	}
}

func main() {
    f1 := f(0)
	fmt.Println(f1())   // 1
	fmt.Println(f1())   // 2
	f2 := f(10)
	fmt.Println(f2())   // 11
	fmt.Println(f2())   // 12
}
```

可以看出，闭包的外层函数就相当于一个对象的构造函数，内层函数绑定在一个隐藏对象上执行并把状态记录到这个隐藏对象。

# 原理

- Go 中的变量是在栈上还是堆上创建，需要编译期通过**逃逸分析**决定，而闭包中引用的变量由于被内层和外层函数同时用到，所以是在堆上创建的
- 看起来是普通的变量操作，在编译时实际上会转为指针间接操作，比如上面代码被改为：
  ```Go
    func f(i int) func() int {
        pI := &i
        return func() int {
            (*pI)++
            return (*pI)
        }
    }
  ```
  本质上类似 C++ 的"引用"类型(自动帮你加指针前的\*)
- 闭包可能用到多个外部环境(外层函数中定义的)变量，所以实际上伴随闭包函数(`f(0)`表达式返回的函数)会同时创建一个匿名结构体对象，
  对象中保存着各个环境变量的指针，形如：
  ```Go
    func f(i int, j int) func() int {
        env := &struct {
            pI *int
            pJ *int
        }{
            pI: &i,
            pJ: &j,
        }
        return func() int {
            (*env.pI)++
            return (*env.pI + *env.pJ)
        }
    }
  ```

# 常见问题

- 创建匿名函数时，不经意就可能引入外部环境变量，导致逻辑错乱，如：
  ```Go
    // 输出所有元素，可以乱序
    func test(sli []int) {
        for i := 0; i < len(sli); i++ {
            go func() {
                fmt.Println(sli[i])
            }()
        }
    }
  ```
  上面不经意间创建了闭包，而`i`本质上是以指针的形式记录在闭包中的，所以最后多个携程可能用到同一个`i`值(取决于携程执行速度)。
  可以这样改进：
  ```Go
    // 输出所有元素，可以乱序
    func test(sli []int) {
        for i := 0; i < len(sli); i++ {
            j := i  // 在循环内部 clone 一个新变量，就可以解决
            go func() {
                fmt.Println(sli[j])
            }()
        }
    }
  ```
