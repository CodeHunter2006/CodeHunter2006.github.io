---
layout: post
title: "PlantUML语法"
date: 2019-11-02 22:00:00 +0800
tags: UML
---

记录 PlantUML 常用语法

# (sequence chart)时序图

`A -> B: func`
A 对象调用 B 对象的 func 函数

`B --> A: return result`
虚线，通常表示调用返回

`autonumber`
为调用自动添加序号

`title xxx_title`
设置一个标题

```
actor A
participant B
database C
```

设置后面使用的角色类型(角色、参与者、数据库)，同时通过从上到下的排列设置从左到右的角色顺序

`A -> B ++/--/**/!!: func`
描述此次调用的生命周期：激活/不激活/创建对象/析构对象

`return [result]`
同步调用的返回[结果]

`note right/left of xxx: this is a note`
在某个角色生命线的左边或右边添加一个注释

`note over AAA, BBB, CCC: this is \n a note`
同时覆盖三个角色生命线的注释

```
note over AAA, BBB
    this is a note
    over AAA and BBB
end note
```

`note over` 语法的另一种格式，可以不使用换行符实现换行

`activate/deactivate A`
激活/不激活 A 的生命周期(开始/结束生命周期)

# 思维导图

`-*+`
分别表示`左中右`位置，后加空格和节点名称，用数量表示层级关系。
