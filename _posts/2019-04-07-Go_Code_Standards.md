---
layout: post
title: "（转）Go编码规范"
date: 2019-04-07 17:40:00 +0800
tags: Go
---

转自：

- [Golang 编码规范](https://segmentfault.com/a/1190000000464394?utm_medium=referral&utm_source=tuicool)
- [Effective Golang](https://golang.org/doc/effective_go.html)
- [Golang Code Review Comments](https://github.com/golang/go/wiki/CodeReviewComments)
- [Uber Go Style Guide](https://github.com/uber-go/guide/blob/master/style.md)
- [Go-advices](https://github.com/cristaloleg/go-advices)

### gofmt

大部分的格式问题可以通过 gofmt 解决，gofmt 自动格式化代码，保证所有的 go 代码一致的格式。

正常情况下，采用 Sublime 或 VSCode 编写 go 代码时，插件已经调用 gofmt 对代码实现了格式化。

- 关于代码中的`;`
  Go 语法解析非常简单，检查每一行是否有结束符，例如`break continue fallthrough return ++ -- ) }`，如果没有结束符则继续解析。
  不需要自己加`;`，语法解析器自己会加上，但是编写代码时要注意格式，通过未结束符来标记实际语义，例如：

```Go
// 正确
if i < f() {
    g()
}

if i < f()  // wrong!
{           // wrong!
    g()
}
```

### 注释

在编码阶段同步写好变量、函数、包注释，注释可以通过 godoc 导出生成文档。

注释必须是完整的句子，以需要注释的内容作为开头，句点作为结尾。

程序中每一个被导出的（大写的）名字，都应该有一个文档注释。

- 包注释

每个程序包都应该有一个包注释，一个位于 package 子句之前的块注释或行注释。

包如果有多个 go 文件，只需要出现在一个 go 文件中即可。

```
//Package regexp implements a simple library
//for regular expressions.
package regexp
```

- 可导出类型

第一条语句应该为一条概括语句，并且使用被声明的名字作为开头。

```
// Compile parses a regular expression and returns, if successful, a Regexp
// object that can be used to match against text.
func Compile(str string) (regexp *Regexp, err error) {
```

### 命名

使用短命名，长名字并不会自动使得事物更易读，文档注释会比格外长的名字更有用。

- 包名

包名应该为小写单词，不要使用下划线或者混合大小写。

- 接口名

单个函数的接口名以"er"作为后缀，如 Reader,Writer

接口的实现则去掉“er”

```
type Reader interface {
        Read(p []byte) (n int, err error)
}
```

两个函数的接口名综合两个函数名

```
type WriteFlusher interface {
    Write([]byte) (int, error)
    Flush() error
}
```

三个以上函数的接口名，类似于结构体名

```
type Car interface {
    Start([]byte)
    Stop() error
    Recover()
}
```

- 混合大小写

采用驼峰式命名

```
MixedCaps 大写开头，可导出
mixedCaps 小写开头，不可导出
```

- 变量

```
全局变量：驼峰式，结合是否可导出确定首字母大小写
参数传递：驼峰式，小写字母开头
局部变量：下划线形式
```

※"局部变量"不同意下划线形式，直接按照参数相同的格式即可。

- Getter and Setter

一般面向对象语言都有自动生成 Getter/Setter 的功能，Go 没有这个功能，由使用者自己实现即可。
Go 的 Getter 不需要在前面加`Get`前缀，直接以大写开头的字段名 Method 就可以做 Getter。Setter 可以和其他语言一样，以`Set`作为前缀。

### 控制结构

- if

if 接受初始化语句，约定如下方式建立局部变量

```
if err := file.Chmod(0664); err != nil {
    return err
}
```

- 如果参与 if 判断的变量本身是 bool 型，则无需跟 false 或 true 进行比较，直接使用变量或"取非!"就可以了

- for

采用短声明建立局部变量

```
sum := 0
for i := 0; i < 10; i++ {
    sum += i
}
```

- range

如果只需要第一项（key），就丢弃第二个：

```
for key := range m {
    if key.expired() {
        delete(m, key)
    }
}
```

如果只需要第二项，则把第一项置为下划线

```
sum := 0
for _, value := range array {
    sum += value
}
```

- return

尽早 return：一旦有错误发生，马上返回

```
f, err := os.Open(name)
if err != nil {
    return err
}
d, err := f.Stat()
if err != nil {
    f.Close()
    return err
}
codeUsing(f, d)
```

### 函数（必须）

- 函数采用命名的多值返回
- 传入变量和返回变量以小写字母开头

```
func nextInt(b []byte, pos int) (value, nextPos int) {
```

在 godoc 生成的文档中，带有返回值的函数声明更利于理解

### 错误处理

- error 作为函数的值返回, 必须对 error 进行处理
- 错误描述如果是英文必须为小写，不需要标点结尾
- 采用独立的错误流进行处理

- 如果失败原因只有一个，则返回 bool 型
- 如果失败原因超过一个，则返回 error 型
- 如果没有失败的情况，则可以不返回 bool 或 error
- 如果重试几次可以避免失败，则不要立即返回 bool 或 error，应该在函数内部进行适当的重试

不要采用这种方式

```Go
if err != nil {
	// error handling
} else {
	// normal code
}
```

而要采用下面的方式

```Go
if err != nil {
	// error handling
	return // or continue, etc.
}
// normal code
```

如果返回值需要初始化，则采用下面的方式

```Go
x, err := f()
if err != nil {
    // error handling
    return
}
// use x
```

### panic

- 尽量不要使用 panic，除非你知道你在做什么

- 在程序开发阶段，坚持速错，让程序尽快 panic 崩溃
- 对于完全意料之外的情况，也尽量使用 error 返回，避免使用 panic。但是程序难以避免还是会发生 panic
- 程序部署后，也尽量不要尝试恢复，可以写 recover 处理尽快打印 log、通知报警，然后退出，不要发生被忽略的情况
- 上面 recover 记录、退出后，由 supervisor 等守护进程重启，可以更好的释放进程内资源
- 如果怕 panic 导致其他线程出问题，可以在 recover 时进行平滑退出处理，尽量完成现有业务动作，不再接收新任务动作

### import

- 对 import 的包进行分组管理，而且标准库作为第一组

```
package main

import (
    "fmt"
    "hash/adler32"
    "os"

    "appengine/user"
    "appengine/foo"

    "code.google.com/p/x/y"
    "github.com/foo/bar"
)
```

`goimports`实现了自动格式化

### 缩写

- 采用全部大写或者全部小写来表示缩写单词

比如对于 url 这个单词，不要使用

```
UrlPony
```

而要使用

```
urlPony 或者 URLPony
```

### 参数传递

- 对于少量数据，不要传递指针
- 对于大量数据的 struct 可以考虑使用指针
- 传入参数是 map，slice，chan 不要传递指针

因为 map，slice，chan 是引用类型，不需要传递指针的指针

### receiver

- 名称

统一采用单字母'p'而不是 this，me 或者 self

```
type T struct{}

func (p *T)Get(){}
```

- 类型

对于 go 初学者，接受者的类型如果不清楚，统一采用指针型

```
func (p *T)Get(){}
```

而不是

```
func (p T)Get(){}
```

在某些情况下，出于性能的考虑，或者类型本来就是引用类型，有一些特例

- 如果接收者是 map,slice 或者 chan，不要用指针传递

```
//Map
package main

import (
    "fmt"
)

type mp map[string]string

func (m mp) Set(k, v string) {
    m[k] = v
}

func main() {
    m := make(mp)
    m.Set("k", "v")
    fmt.Println(m)
}
```

```
//Channel
package main

import (
    "fmt"
)

type ch chan interface{}

func (c ch) Push(i interface{}) {
    c <- i
}

func (c ch) Pop() interface{} {
    return <-c
}

func main() {
    c := make(ch, 1)
    c.Push("i")
    fmt.Println(c.Pop())
}
```

### 返回值命名

- 给返回值命名可以增强程序的可读性性，从名称可推断出用途
- 但是如果中途 return 语句后不跟参数，可能带来新的可读性差的问题
- 所以规定，return 后面还要把返回的变量再写一遍
