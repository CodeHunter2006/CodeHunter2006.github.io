---
layout: post
title: "Go Error Wrapping"
date: 2021-05-29 23:00:00 +0800
tags: Go
---

Go1.13 中引入了 error wraping 机制，可以实现如下功能：

- 类比其他语言的`try catch`机制，可以在 error 判断时匹配特定的 error，这样就可以分层处理 error
- 可以自定义`Is`、`As`接口，实现自定义匹配方式，比如字符串匹配或其他条件
- 可以在自定义 error 中内嵌其他数据，然后在上层再取出

# 基本结构 + 关键接口

```Go
type error interface {
	Error() string
}
```

- Go1.13 提供了 error 的链式结构

```Go
package errors

func Unwrap(err error) error {
	u, ok := err.(interface {
		Unwrap() error
	})
	if !ok {
		return nil
	}
	return u.Unwrap()
}
```

- 通过上面的代码，只要符合`Unwrap`接口，就可以不断 Unwrap 形成 error chain

- 利用 `wraped := fmt.Errorf("test %w", err)`的`%w`就可以对原始 err 进行 wrapping

- 用`errors.Is(wrapedErr, tarErr)`可以判定`tarErr`是否包含在`wrapedErr`中

```Go
var tarErrorPointer *MyError
errors.As(wrapedErr, &tarErrorPointer)
```

- 用上面代码可以断言并取出特定类型的 error，注意声明是是指针类型，传入第二个参数是**指针的地址**

# 用法示例

```Go
type MyError struct {
}

func (p *MyError) Error() string {
	return "origin"
}

func testErrorWrapping() {
	var originErr error = new(MyError)
	fmt.Println(originErr) // origin

	var originErr2 error = errors.New("origin2")

	// 注意这里 %w 只能用一次(靠左)，第二次使用无法起到 wrap 作用
	wrapedError := fmt.Errorf("wrap %w, %v", originErr, originErr2)
	fmt.Println(wrapedError)                // wrap origin
	fmt.Println(errors.Unwrap(wrapedError)) // origin

	isErr := errors.Is(wrapedError, originErr)
	fmt.Println(isErr) // true

	var targetError *MyError
	asErr := errors.As(wrapedError, &targetError)
	fmt.Println(asErr)       // true
	fmt.Println(targetError) // origin
}
```
