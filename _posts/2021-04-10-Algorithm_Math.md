---
layout: post
title: "Algorithm Math"
date: 2021-04-10 22:00:00 +0800
tags: Algorithm Leetcode
---

记录 Math 的算法实现

### "7. Reverse Integer"

```Go
func reverse(x int) (ret int) {
    for x != 0 {
        if ret < math.MinInt32/10 || ret > math.MaxInt32/10 {
            return 0
        }
        single := x%10
        x /= 10
        ret = ret*10 + single
    }
    return ret
}
```

### "264. Ugly Number II" Math+DP

```Go
func nthUglyNumber(n int) int {
    dp := make([]int, n+1)
    dp[1] = 1
    p2, p3, p5 := 1, 1, 1
    for i := 2; i <= n; i++ {
        next2, next3, next5 := dp[p2]*2, dp[p3]*3, dp[p5]*5
        dp[i] = min(min(next2, next3), next5)
        if dp[i] == next2 {
            p2++
        }
        if dp[i] == next3 {
            p3++
        }
        if dp[i] == next5 {
            p5++
        }
    }
    return dp[n]
}
```

### "633. Sum of Square Numbers"

```Go
func judgeSquareSum(c int) bool {
    l, r := 0, int(math.Sqrt(float64(c)))
    for l <= r {
        tmp := l*l + r*r
        if tmp == c {
            return true
        } else if tmp > c {
            r--
        } else {
            l++
        }
    }
    return false
}
```
