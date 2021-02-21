---
layout: post
title: "Algorithm Tips"
date: 2020-12-20 21:00:00 +0800
tags: Algorithm Leetcode
---

![Algorithm_Tips](/assets/images/2020-12-20-Algorithm_Tips_1.png)
记录常用算法技巧

# 数据结构

## Queue(队列)

符合 FIFO(先入先出)，一般从队尾入队，队首出队

### DQueue(double queue，双向队列、双端队列)

dq 可以同时提供两个 queue，一个 queue 的队首是另一个的队尾

- C++ STL 的 deque 实现比较好，底层用多段数组实现
- 也可以用 linkList 代替，只要使用 linkList 操作接口子集就可以
- 用 C++的 vector 或 Go 的 slice 也可以实现，但是队首出队后空间会被浪费

#### dq 应用：Monotone Queue(单调队列)

单调队列用 dq 实现，队列中的元素始终保持有序，队首始终保持最值。
队首元素按顺序出队、队尾元素可以在遍历过程中入队或出队来维护队列元素值有序。

示例：
["1438. Longest Continuous Subarray With Absolute Diff Less Than or Equal to Limit" SlidingWindow+MonotoneQueue Golang]()

# Tips

### **剪枝**

在做某种可能性空间搜索时，如果某一状态以前已经遍历过，可以直接取得这个状态的结果，避免重复计算。
这就好像遍历树的时候，对一些已知肯定没有结果的支杈进行剪枝一样。
通常可以用 map 把状态结果保存，可以快速判断和返回。

### inplace(原地)思想

- 利用数组位置
  有时题目给定的数组元素只需要遍历一遍，这时可以利用原数组位置存储遍历结果以便后面使用。这样可以节省空间，对逻辑也没有干扰。

  - 示例：leetcode "448. Find All Numbers Disappeared in an Array"

- 利用 bit 位
  有时题目给定的数组元素有效位并不多，比如元素是 int32 型而表示的数范围是 1byte 内，那么实际上有将近 3byte 是可以利用的空间，可以存储一些中间结果。
  - 好处：
    1. 减少空间复杂度，直接利用原有输入参数的空间
    2. 不需要考虑映射关系，直接在原地操作即可
  - 坏处：
    1. 如果原数据和新数据边界不好划定，容易出错

### 数组中间元素下标

设数组的长度为 n，如果 n 为奇数，则中间元素下标是`n/2`，经过整数向下取整，正好为中间下标；n 为偶数，则中间下标是`(n/2)-1`。

可以用统一公式`(n-1)/2`表示中间位置。

### 乘/除 2

用`x>>1`代替`x /= 2`可以节省 CPU 周期提高效率

### 判断奇偶性

用`x & 1 == 1`代替`x % 2 != 0`。因为位操作 CPU 效率远远大于取模。

# Golang Tips

- LeetCode 可以用`fmt`输出调试，无法用`log`输出
- Golang 中缺少一些常用的常量定义，可以自己用位操作实现
  - `const UINT_MIN uint = 0` // 所有位都是 0
  - `const UINT_MAX = ^uint(0)` // 所有位都是 1
  - `const INT_MAX = int(^uint(0) >> 1)` // 第一位是 0，其余都是 1
  - `const INT_MIN = ^INT_MAX` // 第一位是 1，其余都是 0
- 可以自己实现常用的 swap 函数

  ```Go
  func swap(a, b *[]int)  {
  	*a, *b = *b, *a
  }
  ```

- 自己实现 max/min 函数
  ```Go
  func max(a, b int) int {
    if a > b {  // 注意这里不用 >= 符号，没有必要
       return a
    }
    return b
  }
  ```
