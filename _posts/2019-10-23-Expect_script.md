---
layout: post
title: "Expect script"
date: 2019-10-23 23:00:00 +0800
tags: Linux
---

![Expect](/assets/images/2019-10-23-Expect_script_1.jpg)

# Expect 是做什么的？

在 unix 系统中以子进程的形式执行各种 shell 命令，可以方便的执行一系列自动化动作。Expect 本身也是一种脚本语言，其内置的命令和语法可以方便的实现这个领域内的功能。

- 在 mac 上要先执行`brew install expect`

# 系统级命令

## expect

执行 expect 脚本，自动模拟人的识别(正则匹配)和输入。

## autoexpect

可以通过互动的方式自动生成`script.exp`脚本文件。

- 通常可以自动方式生成后，分析其中记录的交互过程，再手动整理成最终的脚本。

# 语法/内部命令

## spawn

以子进程(产卵)方式执行命令，这样当前进程不会被阻塞，可以根据反馈走不同分支。

## expect

根据子进程返回的不同文本输出走不同的分支。

## send

向子进程发送文本输入

## interact

xxx

## set

定义变量

## read / put

读入、输出文本

# expect 互动脚本示例

一段需要人互动的 shell 脚本示例：`questions.sh`

```shell
#!/bin/bash
echo "Hello, who are you?"

read $REPLY

echo "Can I ask you some questions?"

read $REPLY

echo "What is your favorite topic?"

read $REPLY
```

与上面对应的一段 expect 脚本示例：`answer.exp`

```shell
#!/usr/bin/expect -f

set timeout -1

spawn ./questions.sh

expect "Hello, who are you?\r"

send -- "Im Adam\r"

expect "Can I ask you some questions?\r"

send -- "Sure\r"

expect "What is your favorite topic?\r"

send -- "Technology\r"

expect eof
```

执行下面命令，用`answer.exp`应对`questions.sh`的问题。

`expect -f answer.exp`

结果如下：

```
spawn ./questions.sh
Hello, who are you?
Im Adam
Can I ask you some questions?
Sure
What is your favorite topic?
Technology
```

- 上面脚本文件开头的文字`#!/usr/bin/expect -f`是指定执行脚本的程序，直接以可执行程序的方式执行脚本时，将用到这些信息
- 文件的后缀名没有必要，只是使用习惯
- 如果要以可执行程序的方式执行脚本，需要修改文件的权限`chmod +x ./answer.exp`
