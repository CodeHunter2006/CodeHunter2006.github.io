---
layout: post
title: "Leetcode Recur"
date: 2020-07-03 23:00:00 +0800
tags: Algorithm Leetcode
---

## 剑指 Offer 62. 圆圈中最后剩下的数字

[题目](https://leetcode-cn.com/problems/yuan-quan-zhong-zui-hou-sheng-xia-de-shu-zi-lcof/)、官方解答[数学 + 递归](https://leetcode-cn.com/problems/yuan-quan-zhong-zui-hou-sheng-xia-de-shu-zi-lcof/solution/yuan-quan-zhong-zui-hou-sheng-xia-de-shu-zi-by-lee/)

- 直观思路
  以输入的参数构建一个链表结构，按照题目步骤要求逐个删除并打印节点
  - 存在的问题
    n 较大时需要构建的节点较多，链表在堆上构建时间较长，删除也比较耗时；m 较大时，每次删除一个节点需要遍历多个节点，链表遍历速度慢。最终导致 n m 较大时性能太差，无法通过 Case。
- 改进思路 1
  当前序列长度为 n，假设 n-1 序列长度下最终留下的为 x，则有函数`x = f(n-1, m)`。设 n-1 序列下，坐标原点为 0，则在 n 序列下，n-1 序列的坐标原点为 m(假设 m < n)，因为 n 序列下删除了 m 位置后，才开始处理 n-1 序列的。所以计算剩余 x 的过程可以转变为横坐标变换过程，`f(n, m) = (m % n + x) % n = (m + x) % n`，图例参考官方解答。
  - 存在的问题
    相比前面的方法，每层关系在栈上维护，相比堆效率更高；没有了节点遍历过程效率较高。但是规模较大时，栈会很大，可能导致大多数语言的栈溢出，所以对于栈规模有限的语言需要用迭代方法改进。
- 改进思路 2
  上面递归方法的迭代实现，空间复杂度从 O(n)优化到 O(1)

```Go
// 改进思路 1
func lastRemaining(n int, m int) int {
    if n == 1 {
        return 0
    }
    x := lastRemaining(n - 1, m)
    return (m + x) % n
}

// 改进思路2
func lastRemaining(n int, m int) int {
    x := 0
    for i := 2; i <= n; i++ {
        x = (x + m) % i
    }
    return x
}
```
