---
layout: post
title: "Algorithm Gemoetry"
date: 2021-07-31 09:00:00 +0800
tags: Algorithm LeetCode
---

记录 Gemoetry 的算法实现

### "149. Max Points on a Line"

```Go
func maxPoints(points [][]int) (ret int) {
    n := len(points)
    const BaseX = (1e4 * 2) + 1
    if n < 3 {
        return n
    }
    for i, p := range points {
        if ret >= n-i || ret > n/2 {
            break
        }
        cnt := make(map[int]int)
        for _, q := range points[i+1:] {
            x, y := p[0]-q[0], p[1]-q[1]
            if x == 0 {
                y = 1
            } else if y == 0 {
                x = 1
            } else {
                if y < 0 {
                    x, y = -x, -y
                }
                g := gcd(abs(x), abs(y))
                x, y = x/g, y/g
            }
            cnt[y + x*BaseX]++
        }
        for _, c := range cnt {
            ret = max(ret, c+1)
        }
    }
    return ret
}

func gcd(a, b int) int {
    for a > 0 {
        a, b = b%a, a
    }
    return b
}
```