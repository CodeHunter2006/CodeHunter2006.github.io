---
layout: post
title: "设计模式——Functional Options Pattern"
date: 2019-10-27 12:00:00 +0800
tags: Go DesignPattern
---

![Functional Options Pattern](/assets/images/2019-10-27-Functional_Options_Pattern_1.png)
Uber 最近开源了内部的编码规范"Uber Go Style"，在阅读中看到这个优雅的设计模式，记录一下。

- 分类：创建型
- 功能：利用不定数量、不定类型的参数作为可选项传入构造函数或普通函数，优雅的实现选项设置、默认值功能
- 备注：如果语言本身支持参数默认值，则直接用默值即可；只如果像 Go 这样没有默认值但有不定数量参数的语言，可以用这个模式实现；如果即没有参数默认值也没有不定数量参数，可以用 Option 数组替代。

## 问题场景

有一个类，在构造时有多个可选参数。通常为了兼容所有的可选参数，我们要让每个可选项都在参数有一个占位，构造时如果不需要这个选项则传一个默认值，如下：

```go
// package db

func Connect(
  addr string,
  timeout time.Duration,
  caching bool,
) (*Connection, error) {
  // ...
}

// Timeout and caching must always be provided,
// even if the user wants to use the default.

db.Connect(addr, db.DefaultTimeout, db.DefaultCaching)
db.Connect(addr, newTimeout, db.DefaultCaching)
db.Connect(addr, db.DefaultTimeout, false /* caching */)
db.Connect(addr, newTimeout, false /* caching */)
```

## 模式示例

```go
type options struct {
  timeout time.Duration
  caching bool
}

// Option overrides behavior of Connect.
type Option interface {
  apply(*options)
}

type optionFunc func(*options)

func (f optionFunc) apply(o *options) {
  f(o)
}

func WithTimeout(t time.Duration) Option {
  return optionFunc(func(o *options) {
    o.timeout = t
  })
}

func WithCaching(cache bool) Option {
  return optionFunc(func(o *options) {
    o.caching = cache
  })
}

// Connect creates a connection.
func Connect(
  addr string,
  opts ...Option,
) (*Connection, error) {
  options := options{
    timeout: defaultTimeout,
    caching: defaultCaching,
  }

  for _, o := range opts {
    o.apply(&options)
  }

  // ...
}

// Options must be provided only if needed.

db.Connect(addr)
db.Connect(addr, db.WithTimeout(newTimeout))
db.Connect(addr, db.WithCaching(false))
db.Connect(
  addr,
  db.WithCaching(false),
  db.WithTimeout(newTimeout),
)
```
