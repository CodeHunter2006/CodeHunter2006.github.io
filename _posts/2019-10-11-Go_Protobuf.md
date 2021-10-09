---
layout: post
title: "Protobuf在Go中的用法介绍"
date: 2019-10-11 22:20:00 +0800
tags: Go Protobuf
---

![protobuf](/assets/images/2019-10-11-Go_Protobuf_1.png)
Protocol Buffer 谷歌开发的开源协议，用于在网络间传输对象，适用于高并发服务端程序开发。

# 序列化时要考虑的性能参数：

- 序列化后的容量，占用网络流量
- 序列化/反序列化的速度，占用 CPU 情况(主要是服务端)
- 是否支持跨语言，便于在各端通信

# protobuf 优点：

- 与 json、xml 相比序列化容量小几倍
- 序列化/反序列化速度快几倍
- 支持跨平台、跨语言
- 维护简单，只要一份协议文件
- 向后兼容性好（新增字段时兼容旧数据结构，只要字段序列号不同即可）
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

- 在 .proto 文件所在的文件夹下执行命令 `protoc --proto_path . --go_out . xxx/xxx/*.proto`，就可以生成了
  - `xxx/xxx/*.proto` 表示哪些文件将被编译
  - `--proto_path` 可以指定去什么路径寻找`import`所需的文件。这个参数可以传入多次。
  - `--go_out` 表示`go`语言对应的文件生成在什么路径下，`.`表示`*.proto`文件所在路径。其他语言有对应的参数。

# .proto 文件语法

| 关键字   | 表示的类型   | 默认值   |
| -------- | ------------ | -------- |
| bool     | -            | false    |
| int32    | -            | 0        |
| int64    | -            | 0        |
| uint32   | -            | 0        |
| sint32   | 负数效率更高 | 0        |
| float    | -            | 0.0      |
| double   | -            | 0.0      |
| string   | -            | ""       |
| enum     | 枚举类型     | proto3:0 |
| message  | 结构体       | 空指针等 |
| repeated | 数组         | 空数组   |

- 在不赋值的情况下，会根据不同类型自动赋默认值
- string 包含的字符数不大于 2^32
- message 的默认值具体随语言而定，在 Go 中是 nil 指针

## 限定符

- required 表示该字段是必须的否则消息无法发送
- optional 表示该字段是可选的，如果不填则自动填写默认值
- repeated 表示 0 ～ n 个元素组成的数组，由于包含 0，所以与 optional 一样是可选的

## tag

message 结构体中，字段右边的数字叫做 tag，表示该字段的唯一标识。在二进制传输时，前面的字段名并不会被传输，只有这个 tag 会被传输。也可以把 tag 看作是字段的索引，这样减少传输容量。

- tag 一般从 1 开始
- tag 的范围是[1, (2^29)-1]，其中 19000~19999 是预留的，开发者不能使用
- 1~15 的 tag 在数据中只占 1 个字节的空间，应该用在最频繁使用的字段上；从 16~2047 占用两个字节，可以用在不频繁使用的字段上

## 保留字段

如果对 proto 文件修改，删除某些字段，这时要注意这些 tag 不能再使用了。因为以前生成的代码或分发出的 proto 文件已经包含了这个字段，而通信时将使用该 tag，同一个 tag 会被认为是同一字段的值，造成不一致的问题。

- 可以在 tag 前加`reserved`关键字，表示预留，这样有别的字段重用了相同 tag 时就会编译报错
- 如果有些字段名在 JSON 序列化时使用了，同样可以标记为 reserved，避免相同字段名重用

## 枚举类型

枚举的本质是一个命名空间(枚举类型名) string->int 的映射关系。

- 枚举的常量值不能超过 int32 范围，同时不建议使用负数
- 枚举可以定义在 message 里作为内嵌枚举类型，也可以在外面单独定义。如果另一个 message 想使用枚举，可以用命名空间`.`的形式引用，比如`Person.Sex`。
- 在枚举中可以 reserved 数值、枚举名称

### 枚举值别名

枚举定义时增加`option allow_alias = true;`，可以设置别名，即多个枚举名称对应一个值。

## import 文件

类似 C++的 include，可以导入另一个文件，然后使用另一个文件中定义的 message 等

## 命名空间、package

一般情况下，可以通过 message **嵌套** message/enum 实现命名空间功能。

如果存在多个文件中定义了相同 message 的情况，这时就需要用`package`指定所属命名空间，这样可以避免 message 名冲突。

- 在使用`package`语法指定命名空间时，可以用`option go_package = "pb"`这种形式单独指定某种语言的命名空间

## extend 字段

有些情况下，要对原有的 message 类型进行扩展，相当于抽象类预留了一些虚函数给子类。

```proto
// 原有 message 定义
package test;
message base {
    optional uint64 timestamp = 1;
    extensions 100 to 1000; // base 要明确提前预留的 tag 范围
}
```

```proto
// 新的扩展实现
package test;
import "main.proto";
extend base {
    optional string my_name = 100;  // 这里的 tag 要在预留范围内
}
```

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
    option allow_alias = true;
    WOMAN = 0;
    MAN = 1;
    MALE = 1;
}

// 每个字段后面都要有数字，表示该字段的唯一标志，可用于版本变化后向后兼容
message Role {
    string roleName = 1 [default = "no role"]; // 可以这样设置默认值
    string place = 2;

    reserved 4, 5;  // 保留 tag
    reserved 20 to 30;  // 保留一段 tag
    reserved 200 to max;    // 保留 200 到最大值
    reserved "name1", "name2";  // 保留字段名
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

# proto3 的语法变化

## 语法标记

如果没有`syntax`定义，默认是`proto2`，如果用`proto3`需增加完整定义`syntax = "proto3";`

## 默认 optional

只保留 repeated 表示数组类型，optional 和 required 都被去掉了，默认就是 optional。由于 required 会造成兼容问题，被废弃。

## 支持 map 数据类型

`map<key_type, value_type> map_field = N;`
这样数据传输更方便了

## 不再支持 default 语法

由于序列化、反序列化的两端对 default 描述可能不一致，会造成最终结果的不一致，所以废弃。

## 枚举默认值一定是 0

proto2 里的默认值是枚举的第一个 value 对应的值, 不一定为 0；proto3 在你定义 value 时, 强制要求第一个值必须为 0。
同样可以避免不一致的隐患。

## 新增不确定类型 any

可以代表任何类型, 可以先读进来, 再进行解析。

## 支持 json 序列化

# 其他注意问题

- 如果必须用 proto2，最好用 proto3 的最佳实践，这样未来容易迁移，也避免问题
- `required` 可能会造成兼容问题，使用时要小心。最好都用`optional`

# protobuf 常见问题

- 问题描述：
  server 端发送数据 client 端缺少新增字段
- 分析过程：
  将二进制内容打印，变更其中一个临近字段值，发现后面的字段被客户端截断了。原因是客户端下载了最新文件后，只对包含了变更字段的文件进行了编译，导致其外层对象没有更新到最新版本，长度缺失
- 解决方案：
  客户端下载最新全部 proto 文件，整体编译生成 .c 文件。以后注意更新文件后，要整体编译。
