---
layout: post
title: "Algorithm Random"
date: 2021-05-04 09:00:00 +0800
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

### "478. Generate Random Point in a Circle"

```Go
import "math/rand"

type Solution struct {
    radius, x_center, y_center float64
}

func Constructor(radius float64, x_center float64, y_center float64) Solution {
    return Solution{
        radius:radius,
        x_center:x_center,
        y_center:y_center,
    }
}

func (this *Solution) RandPoint() (ret []float64) {
    for {
        x, y := rand.Float64()*2.0*this.radius, rand.Float64()*2.0*this.radius
        if (x-this.radius)*(x-this.radius) + (y-this.radius)*(y-this.radius) <= this.radius * this.radius {
            ret = append(ret, this.x_center+x-this.radius, this.y_center+y-this.radius)
            break
        }
    }
    return ret
}
```

### "497. Random Point in Non-overlapping Rectangles"

```Go

```

### "528. Random Pick with Weight"

```Go
import "math/rand"

type Solution struct {
    preSum []int
}

func Constructor(w []int) Solution {
    rand.Seed(time.Now().Unix())

    preSum := make([]int, len(w))
    for i, sum := 0, 0; i < len(w); i++ {
        sum += w[i]
        preSum[i] = sum
    }

    return Solution{
        preSum: preSum,
    }
}

func (this *Solution) PickIndex() int {
    target := rand.Intn(this.preSum[len(this.preSum)-1])
    l, r := -1, len(this.preSum)
    for l + 1 != r {
        mid := (l+r)>>1
        if this.preSum[mid] <= target {
            l = mid
        } else {
            r = mid
        }
    }
    return l+1
}
```
