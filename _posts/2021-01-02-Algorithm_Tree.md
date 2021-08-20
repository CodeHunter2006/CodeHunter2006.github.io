---
layout: post
title: "Algorithm Tree"
date: 2021-01-02 09:00:00 +0800
tags: Algorithm Leetcode
---

记录 Tree 的算法实现

### "124. Binary Tree Maximum Path Sum"

```Go
var gMaxNum int
func maxPathSum(root *TreeNode) (ret int) {
    gMaxNum = math.MinInt32
    var dfs func(*TreeNode) int
    dfs = func(cur *TreeNode) (ret int) {
        if cur == nil { return 0 }
        l, r := dfs(cur.Left), dfs(cur.Right)
        ret = max(cur.Val, l+cur.Val, r+cur.Val)
        gMaxNum = max(gMaxNum, ret, l+r+cur.Val)
        return ret
    }
    dfs(root)
    return gMaxNum
}
```

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

### "863. All Nodes Distance K in Binary Tree"

```Go
func distanceK(root *TreeNode, target *TreeNode, k int) (ret []int) {
   var dfs func(*TreeNode, bool, int) (bool, int)
   dfs = func(cur *TreeNode, inFound bool, inSteps int) (outFound bool, outSteps int) {
       if cur == nil {
           return false, 0
       }
       if inFound {
           if inSteps == k {
               ret = append(ret, cur.Val)
               return false, 0
           }
           dfs(cur.Left, inFound, inSteps+1)
           dfs(cur.Right, inFound, inSteps+1)
       } else {
           if cur == target {
                dfs(cur, true, 0)
                return true, 1
           }
           lF, lS := dfs(cur.Left, false, 0)
           if lF {
               if lS == k {
                   ret = append(ret, cur.Val)
               }
               if lS < k {
                   dfs(cur.Right, true, lS+1)
               }
               return true, lS+1
           }
           rF, rS := dfs(cur.Right, false, 0)
           if rF {
               if rS == k {
                   ret = append(ret, cur.Val)
               }
               if rS < k {
                   dfs(cur.Left, true, rS+1)
               }
               return true, rS+1
           }
       }
       return false, 0
   }
   dfs(root, false, 0)
   return ret
}
```

### "987. Vertical Order Traversal of a Binary Tree"

```Go
func verticalTraversal(root *TreeNode) (ret [][]int) {
    type nodeInfo struct{
        Row, Col, Val int
    }
    var sli []*nodeInfo

    var dfs func(*TreeNode, int, int)
    dfs = func(cur *TreeNode, row, col int) {
        if cur == nil {
            return
        }
        sli = append(sli, &nodeInfo{
            Row: row,
            Col: col,
            Val: cur.Val,
        })
        dfs(cur.Left, row+1, col-1)
        dfs(cur.Right, row+1, col+1)
    }
    dfs(root, 0, 0)

    sort.Slice(sli, func(a, b int) bool {
        if sli[a].Col != sli[b].Col {
            return sli[a].Col < sli[b].Col
        }
        if sli[a].Row != sli[b].Row {
            return sli[a].Row < sli[b].Row
        }
        return sli[a].Val < sli[b].Val
    })

    lastCol := sli[0].Col - 1
    for _, node := range sli {
        if node.Col != lastCol {
            ret = append(ret, make([]int, 0, 1))
            lastCol = node.Col
        }
        l := len(ret)
        ret[l-1] = append(ret[l-1], node.Val)
    }
    return ret
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
