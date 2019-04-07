---
layout: post
title:  "（转）Go编码标准"
date:   2019-04-07 17:40:00 +0800
tags: Go
---
转自：[https://segmentfault.com/a/1190000000464394?utm_medium=referral&utm_source=tuicool](https://segmentfault.com/a/1190000000464394?utm_medium=referral&utm_source=tuicool)

### gofmt
大部分的格式问题可以通过gofmt解决，gofmt自动格式化代码，保证所有的go代码一致的格式。

正常情况下，采用Sublime编写go代码时，插件GoSublilme已经调用gofmt对代码实现了格式化。

### 注释
在编码阶段同步写好变量、函数、包注释，注释可以通过godoc导出生成文档。

注释必须是完整的句子，以需要注释的内容作为开头，句点作为结尾。

程序中每一个被导出的（大写的）名字，都应该有一个文档注释。

* 包注释

每个程序包都应该有一个包注释，一个位于package子句之前的块注释或行注释。

包如果有多个go文件，只需要出现在一个go文件中即可。
```
//Package regexp implements a simple library 
//for regular expressions.
package regexp 
```
* 可导出类型

第一条语句应该为一条概括语句，并且使用被声明的名字作为开头。
```
// Compile parses a regular expression and returns, if successful, a Regexp
// object that can be used to match against text.
func Compile(str string) (regexp *Regexp, err error) {
```