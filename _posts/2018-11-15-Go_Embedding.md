---
layout: post
title:  "Go基础学习 Go的匿名组合机制"
date:   2018-11-15 10:00:00 +0800
tags: Go
---
### Go中的面向对象实现
在之前使用的面向对象语言(C++、Java、C#、Ruby)中，OO(面向对象)的实现方式通常是：封装+继承+多态。<br/>
在Go中，通过大小写实现封装；抛弃了"继承"和"多态"，而采用"匿名组合"的方式。
```
type Base1 struct {}
func (p *Base1) Func1() {
	fmt.Println("Base1.Func1")
}
func (p *Base1) Func2() {
	fmt.Println("Base1.Func2")
}
func (p *Base1) Func3() {
	fmt.Println("Base1.Func3")
}

type Base2 struct {}
func (p *Base2) Func2() {
	fmt.Println("Base2.Func2")
}

type Derived struct {
	Base1
	Base2
}
func (p *Derived) Func3() {
	fmt.Println("Derived.Func3")
}
func (p *Derived) Func4() {
	fmt.Println("Derived.Func4")
}

func main () {
	pDerived := new(Derived)
	pDerived.Func1()	// Base1.Func1
	//pDerived.Func2()	存在歧义，无法编译通过
	pDerived.Base2.Func2()	// 明确了函数调用对象 Base2.Func2
	pDerived.Func3()	// Derived.Func3
	pDerived.Func4()	// Derived.Func4
}
```
*PS: 由于不存在多态，在基类函数中调用函数，调用的一定是基类函数。*

<br/>
### 优点
* 内存结构清晰，可以直接看懂组合关系
* 在多继承情况下，如果存在相同基类函数被调用的情况，会直接编译不过，不会出现隐含错误
