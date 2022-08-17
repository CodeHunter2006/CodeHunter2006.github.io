---
layout: post
title: "Go 处理 error、panic 的一些最佳实践"
date: 2020-03-22 22:00:00 +0800
tags: Go
---

![InfluxDB + Grafana](/assets/images/2020-03-22-Go_error_best_practice_1.png)
**error** 表示我们可预见的错误，例如"错误的参数范围、函数返回 error、超时..."，可以在代码里提前写好处理代码；**panic** 是我们无法预料的异常，例如"除 0、空指针引用、下标越界..."，通常在主函数、调用库的函数中可能用到 recover 来恢复，或者干脆让程序退出。

# error 处理的一些设计点

- **尽量多用**通常每个函数调用后的返回值都有 error 类型，要先判断这个值再使用其他返回值
- **error 位置**通常每个函数返回值中，最后一个(最右边)是`error`类型，如果没有错误返回值是 nil
- **作用范围**在`if`语句中，用`if err := funcXXX(); err != nil {`这样可以限制`err`变量的作用范围在`if`之内，执行后立刻判断也能节省一行代码。注意多返回值情况下其他变量可能被覆盖的错误。
- **无需判断**对于有些情况，即使被调用的函数返回了 error，调用者也并不关心，这时就没必要非得判断 error
- **善后处理**由于函数中有多处`error`判断，可能随时`return`，所以通常利用`defer`进行"善后"处理，避免每处返回都需要处理
- **无 error**有些函数的定义，里面没有什么可产生的 error，这时没必要非要加个 error 返回值，因为一旦加上，调用者又得多出 error 处理语句
- **error 常量**对于要区分具体 error 类型的情况，可以在包内添加`var _ERROR_XXX = errors.New("xxx")`这样的全局用 error 变量，然后在需要时通过向外部暴露的函数返回出去，再增加判断函数进行比较，具体可参考`os.IsNotExist(err)`的用法
  - 这里的变量名全大写，表示是个常量(实际上是个变量，但是约定不要修改)
  - 变量名前边的`_`表示这个变量不导出，在包内部使用
  - 由于变量未导出，所以做比较时候要用函数判断
  - error 是一个接口类型，所以不要对两个 error 变量进行比较，除非接口内部的值值指针指向相同对象
- **日志输出**通常返回 error 的情况下就不需要输出日志了，只有在主函数或根 goroutine 中才将 error 输出为日志，避免重复。如果输出日志是为了 debug 则不受这条规则影响。在输出日志时，如果可能存在多个携程运行的情况，应该加上 TraceId，它是在携程启动或每个独立循环开头处创建的 UUID
- **bool 替代 error**有些函数只关心结果、不关心是否有错误，可以直接返回 bool 型而不是用"nil"代替"true"。在函数中，由于存在未上报的 error，所以要打印 log。
- **error 格式**在每此调用发生 error 要向上返回时要加上当前所处错误信息，并使信息可以连接起来表达调用栈关系。例如在一次调用文件操作时得到一个 error 返回值，向上抛时用`return fmt.Errorf("file operation error: %v", err)`。这样在主函数中将得到这样的错误信息`"main function call 1 error: file operation error: can not create file"`，这就很容易通过 error 信息定位调用栈及出错点。通常用`:`作为不同层级 error 信息的分隔符。注意每层的错误信息中不要以符号结尾，因为本层的错误信息可能前后都有别的层的错误信息，增加结尾符号可能难以理解。
  - 可以利用`runtime.Caller`直接获取栈帧信息向上返回，避免重复人工输入
- **重试处理**如果在本层可以通过多次重试解决，那就不要直接上抛 error，这样避免重试逻辑涉及过多层级增加了复杂度

# 异常处理的一些设计点

- **不要使用**panic 相比 error 潜在风险要高很多，尤其对于大型服务端程序来说，尽量不用或少用
- **初期试错**在程序开发初期，不用 recover，尽快让它"死给你看"，由于 panic 信息里包含调用栈，可以快速定位问题
- **谨慎 recover**在服务上线后，可以考虑在主函数中用 recover 记录出错信息然后继续主循环，避免服务下线
  - 但是这种方式有潜在风险，你可能长期没有注意一个 panic，而它可能引起以后更大的问题，还不如早一点死掉被发现
  - 或者 panic 的原因是资源占满(端口占满、内存占满)，而 recover 后资源仍然未释放，导致服务变成一个"僵尸服务"，不报警、也不能提供服务
  - 可以直接 panic 死掉，然后用 supervisor 这种进程管理工具自动重启，这样比自己 recover 更好
  - 可以结合 webhook 报警和 supervisor，在主函数 recover 中记录环境、出错信息，发到 webhook，然后程序退出，等待 supervisor 彻底重启
- **谨慎发起 panic**某些严重情况需要主动发起 panic，比如某个资源在服务初始化可用条件下服务才运行，但是运行中发生了未知的无法使用的情况，这时候可以直接发起 panic 但是一定要在文档中说明，或者干脆用 error 替代
- **组件不该 panic**作为被其他程序调用的组件，尽量以 error 返回错误，因为你写的组件可能被别人非常重要的服务调用，随便 panic 可能引发严重问题
  - 如果某个接口的文档说的很明白，就是可能 panic 的，那就可以
  - 同样的在调用未知组件时候，如果对其质量不放心，也要加 recover 防止其意外的 panic，然后把 recover 捕获的 panic 信息抽取为 error 上报
- **子协程 panic**子协程 panic 时，如果不进行 recover 处理，那么整个程序会退出，导致包括主协程在内的其他协程全部退出，所以可能带来一系列风险
  - recover 只能作用于当前的携程调用栈发生 panic，所以无法阻止子携程 panic 导致的程序退出
  - 和组件一样的问题，你不知道你调用的组件是否发起了某个协程而该协程可能会 panic，所以即使你对组件调用函数进行了 recover 防御，还是难以阻止 panic
- **ErrorGroup**如果子携程需要返回错误，可以通过封装了 Context 的 ErrorGroup 对象返回错误信息

- **将 panic 封装为 error:**

```Go
// Compile returns a parsed representation of the regular expression.
func Compile(str string) (regexp *Regexp, err error) {
    regexp = new(Regexp)
    // doParse will panic if there is a parse error.
    defer func() {
        if e := recover(); e != nil {
            regexp = nil    // Clear return value.
            err = e.(Error) // Will re-panic if not a parse error.
        }
    }()
    return regexp.doParse(str), nil
}
```

- 上面的类型强转涉及的类型最好在本包中，否则可能引起 Panic
- 可以尝试强转败，就通过 fmt.Errorf("%v", err) 返回新 error，这样意料之外的类型也不会引起再次 Panic 了
