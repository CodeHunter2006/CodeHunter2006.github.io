---
layout: post
title: "GoModules基本使用方法"
date: 2019-10-01 10:00:00 +0800
tags: Go
---

# 环境变量

## GO111MODULE=auto

表示是否采用 Go modules，可选项`auto/on/off`

- auto 项目中包含`go.mod`文件时才启用 Go modules
- on 启用 Go modules
- off 不启用，仍然使用 GOPATH
- 默认值`auto`

## GOPROXY=https://proxy.golang.org,direct

用`,`分割的 Go module proxy 列表

- 如果值填写`off`则不允许 Go 从任何地方拉取版本

国内可选的代理：

- 阿里云 https://mirrors.aliyun.com/goproxy/
- https://goproxy.io/
- 七牛云 https://goproxy.cn
