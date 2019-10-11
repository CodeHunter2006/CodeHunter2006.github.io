---
layout: post
title: "Protobuf在Go中的用法介绍"
date: 2019-10-11 22:20:00 +0800
tags: Go Protobuf
---

![protobuf](/assets/images/2019-10-11-Go_Protobuf_1.png)
protobuf 协议用于在网络间传输对象，适用于高并发服务端程序开发。

# 序列化时要考虑的性能参数：

- 序列化后的容量，占用网络流量
- 序列化/反序列化的速度，占用 CPU 情况(主要是服务端)
- 是否支持跨语言，便于在各端通信

# protobuf 优点：

- 与 json、xml 相比序列化容量小几倍
- 序列化/反序列化速度快几倍
- 支持跨平台、跨语言
- 维护简单，只要一份协议文件
- 向后兼容性好（新增字段时兼容旧数据结构）
- 加密性好，传输使用二进制

# protobuf 缺点：

- 二进制可读性差
- 需要工具生成依赖文件，有些不便
- 通用性差，现在普遍使用 json

# 使用流程

1. 编写 xxx.proto 协议文件，声明对象的字段类型、名称、对象嵌套关系等信息。
2. 通过编译器（protoc）生成 xxx.pb.cc 等各种语言使用的编译文件，其中包含了根据对象成员名称生成的一套易用的 API(getter、setter)。
3. 引用生成的文件，直接利用 API 进行对象的序列化、反序列化。

# Q&A

1. 为什么不用动态的方式(例如反射)处理传输的变量，而要用生成文件的方式？

- json 等文本格式在脚本语言(如 Python、Javascript)中传输对象时，可以利用脚本的动态创建结构体的特性，实现对象的反序列化。但是静态语言(如 Go、C++、Java)从语言上是无法实现的，所以要生成编译文件。
- 如果利用反射机制动态处理数据，性能也会下降很厉害，并且还要依赖很多未编译的字符串，性能、安全两方面都有问题，还不如干脆根据规则生成一套 API 来调用。
- 凡事都要看适用的场景，protobuf 适用于大容量序列化、传输、静态语言的场景，可以降低服务端的网络和 CPU 压力。相反的脚本语言、少量数据、服务器压力小的情况下，用 json 的通信方式更便捷。

# 具体安装过程

- 在`https://github.com/google/protobuf/releases`下载合适的版本，将其中的可执行文件`protoc`复制到`$GOPATH/bin`

- 安装 protobuf go 插件 `go get -u github.com/golang/protobuf/protoc-gen-go`

- 在 .proto 文件所在的文件夹下执行命令 `protoc --proto_path=./ --go_out=. *.proto`，就可以生成了
  - `--proto_path` 可以指定去什么路径寻找`import`所需的文件。这个参数可以传入多次。
  - `--go_out` 表示`go`语言对应的文件生成在什么路径下。其他语言有对应的参数。

# .proto 文件语法

| 关键字   | 表示的类型   | 默认值 |
| -------- | ------------ | ------ |
| bool     | -            | false  |
| int32    | -            | 0      |
| int64    | -            | 0      |
| uint32   | -            | 0      |
| sint32   | 负数效率更高 | 0      |
| float    | -            | 0.0    |
| double   | -            | 0.0    |
| string   | -            | ""     |
| enum     | 枚举类型     | -      |
| message  | 结构体       | -      |
| repeated | 数组         | -      |

- 在不赋值的情况下，会根据不同类型自动赋默认值

## .proto 示例：

```proto3
// 可以用双斜杠进行注释
// 指定版本，如果为3需要指定，默认为 2
syntax = "proto3"

// 包名，通过 protoc 生成时 go 文件时要用到
package test;

// option 可以添加一些选项
// SPEED(default) 优化序列化、反序列化速度
// CODE_SIZE 优化代码容量
// LITE_RUNTIME 只使用 Message 的子集 MessageLite，速度快
option optimize_for = LITE_RUNTIME;

// 可以导入其他 .proto 文件中的定义
// 注意执行 protoc 命令的当前目录要与 import 相匹配，以免找不到路径
import "xxx/xxx.proto";

// 枚举类型第一个字段序号必须为 0
enum SexType {
    WOMAN = 0;
    MAN = 1;
}

// 每个字段后面都要有数字，表示该字段的唯一标志，可用于版本变化后向后兼容
message Role {
    string roleName = 1 [default = "no role"]; // 可以这样设置默认值
    string place = 2;
}

// 结构体类型 message，可嵌套
message Person {
    required int id = 1;    // required 表示该字段必须设置
    optional string name = 2;   // optional 表示该字段为可选
    SexType sex = 3;    // 可以包含枚举
    repeated Role roles = 4;  // 可以包含数组
}
```

## 使用示例：

```go
package main;

import (
    "github.com/golang/protobuf/proto"
    "protobuf/test"
)

p1 := test.Person {
    id : 1,
    name : "test",
    sex : test.SexType_MAN,
    roles : []*test.Role{
        {roleName: "Manager", place: "Company"},
        {roleName: "Dreamer", place: "Home"}
    },
}

// 编码为字节数组
data, _ := proto.Marshal(p1);

p2 := &test.Person{}
// 解码为对象
proto.Unmarshal(data, p2);
```

# 注意问题

- `required` 可能会造成兼容问题，使用时要小心
