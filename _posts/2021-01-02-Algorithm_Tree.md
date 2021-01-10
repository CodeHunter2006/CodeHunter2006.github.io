---
layout: post
title: "Algorithm Tree"
date: 2021-01-02 09:00:00 +0800
tags: Algorithm Leetcode
---

记录 Tree 的算法实现

### 337. House Robber III" Golang

```Go
func rob(root *TreeNode) int {
    return max(recur(root))
}

func recur(node *TreeNode) (yes, no int) {
    if node == nil {
        return 0, 0
    }
    lY, lN := recur(node.Left)
    rY, rN := recur(node.Right)
    yes = node.Val + lN + rN
    no = max(lY, lN) + max(rY, rN)
    return
}

func max(a, b int) int {
    if a > b {
        return a
    }
    return b
}
```
