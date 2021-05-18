---
layout: post
title: "Algorithm Tree"
date: 2021-01-02 09:00:00 +0800
tags: Algorithm Leetcode
---

记录 Tree 的算法实现

### "337. House Robber III" Golang

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

### "993. Cousins in Binary Tree"

```Go
var gLevel int
var gParent *TreeNode
var gSet map[int]bool

func isCousins(root *TreeNode, x int, y int) bool {
    gLevel, gParent, gSet = math.MaxInt32, nil, map[int]bool{x: true, y: true}
    if recur(root, nil, 0) == 1 {
        return true
    }
    return false
}

// recur return: -1 失败, 0 查找中, 1 都找到
func recur(cur, parent *TreeNode, curLevel int) int {
    if cur == nil || curLevel > gLevel {
        return 0
    }
    if gSet[cur.Val] {
        if gLevel == math.MaxInt32 {
            gLevel, gParent = curLevel, parent
            return 0
        } else if gLevel != curLevel || parent == gParent {
            return -1
        }
        return 1
    }
    if tmp:= recur(cur.Left, cur, curLevel+1); tmp != 0 {
        return tmp
    }

    return recur(cur.Right, cur, curLevel+1)
}
```
