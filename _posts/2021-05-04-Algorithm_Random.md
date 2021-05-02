---
layout: post
title: "Algorithm Math"
date: 2021-04-10 22:00:00 +0800
tags: Algorithm Leetcode
---

记录 Random 的算法实现

### "382. Linked List Random Node"

```Go
import "math/rand"

type Solution struct {
    root *ListNode
}

func Constructor(head *ListNode) Solution {
    rand.Seed(13)
    return Solution{root: head}
}

func (this *Solution) GetRandom() (ret int) {
    for count, cur := 1, this.root; cur != nil; cur, count = cur.Next, count+1 {
        if rand.Float32() < 1.0/float32(count) {
            ret = cur.Val
        }
    }
    return ret
}
```

### "398. Random Pick Index"

```Go
import "math/rand"

type Solution []int

func Constructor(nums []int) Solution {
    return nums
}

func (this *Solution) Pick(target int) (ret int) {
    for count, i := 0, 0; i < len(*this); i++ {
        if (*this)[i] == target {
            if count++; rand.Float32() < 1.0/float32(count) {
                ret = i
            }
        }
    }
    return ret
}
```

### "470. Implement Rand10() Using Rand7()"

- 暴力解法，100 次采样时不通过

```Go
func rand10() int {
   return (rand7()+rand7()+rand7()+rand7()+rand7()+rand7()+rand7())/7
}
```

- 拒绝采样，1000 次采样时不通过

```Go
func rand10() int {
    tmp := 12
    for tmp > 11 {
        tmp = rand7()+rand7()
    }
   return tmp-1
}
```

- 拒绝采样，通过

```Go
func rand10() int {
    idx := 41
    for idx > 40 {
        row, col := rand7(), rand7()
        idx = col+(row-1)*7
    }
    return 1 + (idx-1)%10
}
```
