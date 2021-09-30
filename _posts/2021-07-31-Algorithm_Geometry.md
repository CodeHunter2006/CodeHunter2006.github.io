---
layout: post
title: "Algorithm Gemoetry"
date: 2021-07-31 09:00:00 +0800
tags: Algorithm LeetCode
---

记录 Gemoetry 的算法实现

### "48. Rotate Image"

```Go
func rotate(matrix [][]int)  {
    n := len(matrix)
    for x := 0; x < n>>1; x++ {
        for y := x; y < n-x-1; y++ {
            matrix[x][y], matrix[n-1-y][x], matrix[n-1-x][n-1-y], matrix[y][n-1-x] =
            matrix[n-1-y][x],matrix[n-1-x][n-1-y],matrix[y][n-1-x], matrix[x][y]
        }
    }
}
```

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

### "223. Rectangle Area"

```Go
func computeArea(ax1 int, ay1 int, ax2 int, ay2 int, bx1 int, by1 int, bx2 int, by2 int) int {
    area1 := (ax2-ax1) * (ay2-ay1)
    area2 := (bx2-bx1) * (by2-by1)
    interHeight := min(ay2, by2) - max(ay1, by1)
    interWidth := min(ax2, bx2) - max(ax1, bx1)
    interArea := max(interHeight, 0) * max(interWidth, 0)
    return area1 + area2 - interArea
}
```