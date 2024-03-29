---
layout: post
title: "Algorithm Tree"
date: 2021-01-02 09:00:00 +0800
tags: Algorithm Leetcode
---

记录 Tree 的算法实现

### "109. Convert Sorted List to Binary Search Tree"

```Go
func sortedListToBST(head *ListNode) *TreeNode {
    if head == nil { return nil }
    slow, fast, pre := head, head.Next, head
    for fast != nil {
        pre = slow
        slow = slow.Next
        fast = fast.Next
        if fast != nil {
            fast = fast.Next
        }
    }

    if pre == slow {
        return &TreeNode { Val: slow.Val }
    }

    pre.Next = nil
    next := slow.Next
    slow.Next = nil
    cur := &TreeNode{
        Val : slow.Val,
        Left: sortedListToBST(head),
        Right: sortedListToBST(next),
    }
    return cur
}
```

### "114. Flatten Binary Tree to Linked List"

```Go
func flatten(root *TreeNode)  {
    var dfs func(*TreeNode) *TreeNode
    dfs = func(cur *TreeNode) *TreeNode {
        if cur == nil { return nil }
        fmt.Println(cur.Val)
        tmpRight, tail := cur.Right, cur
        if cur.Left != nil {
            cur.Right = cur.Left
            tail = dfs(cur.Left)
            cur.Left = nil
            tail.Right = tmpRight
        }
        if tmpRight != nil {
            tail = dfs(tmpRight)
        }
        return tail
    }
    dfs(root)
}
```

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

### "437. Path Sum III"

```Go
func pathSum(root *TreeNode, targetSum int) (ret int) {
    m := map[int]int{0: 1}
    var dfs func(*TreeNode, int)
    dfs = func(cur *TreeNode, pre int) {
        if cur == nil { return }
        pre += cur.Val
        ret += m[pre-targetSum]
        m[pre]++
        dfs(cur.Left, pre)
        dfs(cur.Right, pre)
        m[pre]--
        if m[pre] == 0 {
            delete(m, pre)
        }
    }
    dfs(root, 0)

    return ret
}
```

### "440. K-th Smallest in Lexicographical Order"

```Go
func findKthNumber(n int, k int) int {
	getCount := func(prefix int) int {
		cur, next := prefix, prefix + 1
		count := 0
		for cur <= n {
			// 下一个前缀的起点减去当前前缀的起点就是 prefix 开始到 n 之间的本层节点数量
			count += min(next, n+1) - cur
			cur *= 10
			next *= 10
		}
		return count
	}
	preNum := 1
	prefix := 1
	for preNum < k {
		count := getCount(prefix)
		if count+preNum > k {
			// 此时说明满足条件的元素在 prefix 字典树下，转到 prefix 子树下寻找
			prefix *= 10
			// prefix元素的序号是要统计的
			preNum++
		} else {
			// 此时说明在当前这一层的某棵子树下，我们给prefix++，表示寻找下一棵同级子树
			prefix++
			// 并且统计元素数量
			preNum += count
		}
	}
	return prefix
}
```

### "543. Diameter of Binary Tree"

```Go
func diameterOfBinaryTree(root *TreeNode) (ret int) {
    var depth func(*TreeNode) int
    depth = func(cur *TreeNode) int {
        if cur == nil { return 0 }
        l, r := depth(cur.Left), depth(cur.Right)
        ret = max(ret, l+r+1)
        return max(l,r)+1
    }
    depth(root)
    return ret-1
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
