---
layout: post
title: "Go error 处理最佳实践"
date: 2023-06-05 22:00:00 +0800
tags: Go
---

![](/assets/images/2023-06-05-Go_Error_best_practice_1.webp)

在之前[Go 处理 error、panic 的一些最佳实践](/2020/03/22/Go_error_best_practice/)和[Go Error Wrapping](/2021/05/29/Go_Error_Wrap/)的基础上补充最新的 error 处理最佳实践。

# error 上抛的要点：

- error 信息要**便于阅读**，方便快速定位错误，提供有效提示
  - 栈间用`:`分割，栈顶放在最右边。
- error 中最好包含**调用栈**(代码行数信息)，以便调查时快速定位问题
  - 栈信息较多，只在最上层打印一次；并且不能把栈信息直接返给上层服务，避免泄露内部结构
- 上层 error 要对下层 error 进行 wrap，以保证下层 error 信息不会被屏蔽，在上层可以利用`errors.As(wrapedErr, &tarErrorPointer)`提取重要的下层 error
- 对比必要的自定义信息，如**http 调用**时的 errorCode，可以自定义 Error 类型、记录 err.ErrorCode，并在最上层取出
- 对于上层调用无需知道的信息，可以封装一层 Error，并在最上层返回，避免上层需要知道其他组件的内部错误

# github.com/pkg/errors

继承了内置的`errors`包，提供了更便捷的功能

```Go
import "github.com/pkg/errors"

err1 := errors.New("this is an error")  // 创建 error 对象
err := errors.WithMessage(err1, "upper level error") // wrap with message

fmt.Println(err)  // output all error message
// upper level error: this is an error

fmt.Printf("%+v", err) // output call all message and stacks
/*
upper level error: this is an error
main.main
    /usr/three/main.go:11
main.main
    /usr/three/main.go:15
*/
```

- 在`errors.New`、`errors.Errorf`、`errors.Wrap`中包含了`errors.WithStack`调用，适用于在最底层取得调用栈
- 在`errors.WithMessage`中包含了`Wrap`逻辑，包装底层 error 同时增加错误信息，并且信息自动追加到字符串左边
- `errors.Cause`和`errors.Unwrap`实现相同，只是函数名不同
- 打印时`"%v"`可打印基本信息，`"+v"`可打印所有信息

# 在包中定义 const 类型的 error 对象，以便外部用于比较

```Go
const Nil = NilError("nil")

type NilError string
func (e NilError) Error() string { return string(e) }
```
