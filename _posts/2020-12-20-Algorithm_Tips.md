---
layout: post
title: "Algorithm Tips"
date: 2020-12-20 21:00:00 +0800
tags: Algorithm Leetcode
---

![Algorithm_Tips](/assets/images/2020-12-20-Algorithm_Tips_1.png)
记录常用算法技巧

# Tips

### 数组中间元素下标

设数组的长度为 n，如果 n 为奇数，则中间元素下标是`n/2`，经过整数向下取整，正好为中间下标；n 为偶数，则中间下标是`(n/2)-1`。

可以用统一公式`(n-1)/2`表示中间位置。

### 乘/除 2

用`x>>1`代替`x/2`可以节省 CPU 周期提高效率

# Golang Tips

- LeetCode 可以用`fmt`输出调试，无法用`log`输出
- Golang 中缺少一些常用的常量定义，可以自己用位操作实现
  - `const UINT_MIN uint = 0` // 所有位都是 0
  - `const UINT_MAX = ^uint(0)` // 所有位都是 1
  - `const INT_MAX = int(^uint(0) >> 1)` // 第一位是 0，其余都是 1
  - `const INT_MIN = ^INT_MAX` // 第一位是 1，其余都是 0
