---
layout: post
title:  "Go基础学习 string rune"
date:   2018-10-14 17:40:00 +0800
tags: Go
---
<br/>
**len("hello 你好")的迷惑：**<br/>
Go中的string是以UTF-8编码，底层实现是byte数组，想要输出字符串长度时，如果包含了非Ascii：
```
s := string("hello 你好")
fmt.Println(len(s))	// 输出： 12
```
因为len计算了所有byte数量，而一个中文占3byte。

<br/>
**关于rune：**<br/>
想要正确输出字符数量，可以用下面代码：
```
fmt.Println(len([]rune("hello 你好")))	// 输出8
```
这里的rune类型本质上是int32，表示将每个UTF-8字符以int32表示。
Go中单引号一个字符'你'类型就是rune，虽然'你'在UTF-8中占三个字节，但是这里用4个字节表示。
字符串被转化成rune切片后，len就是字符数量了。
* rune的英文读音[run]，意思是"符文"

<br/>
**如何修改string内容？**<br/>
直接按下标操作string元素，本质上是操作的某一个byte，按照上面的说明，并没有操作完整的UTF-8字符。
将常量string转为切片时，会创建一个新的空间存储字符串，可以利用slice作为中间值修改string：
```
s := string("hello 你好")
sli := []rune(s)
sli[0] = '我'
s = string(sli)
fmt.Println(s)	// 输出："我ello 你好"，完整替换了字符
```
