---
layout: post
title: "Algorithm Brute Force"
date: 2020-12-27 09:00:00 +0800
tags: Algorithm Leetcode
---

记录 Brute Force 的算法实现

### "32. Longest Valid Parentheses" Golang

```Go
func longestValidParentheses(s string) int {
    maxLength := 0
    updateMax := func(val int) {
        if val > maxLength {
            maxLength = val
        }
    }

    baseChar := [2]byte{'(', ')'}
    for dir := 0; dir < 2; dir++ {
        base, expect := 0, 0
        for i := 0; i < len(s); i++ {
            c := s[i]
            if dir == 1 {
                c = s[len(s)-1-i]
            }

            if c == baseChar[dir] {
                base++
            } else {
                expect++
            }

            if base == expect {
                updateMax(2 * base)
            } else if expect > base {
                base, expect = 0, 0
            }
        }
    }

    return maxLength
}
```

### "274. H-Index"

```Go
func hIndex(citations []int) (h int) {
    sort.Ints(citations)
    for i := len(citations)-1; i >= 0 && citations[i] > h; i,h = i-1,h+1 {}
    return h
}
```

### "414. Third Maximum Number"

```Go
func thirdMax(nums []int) int {
    var a, b, c *int
    for _, v := range nums {
        v := v
        if a == nil || v > *a {
            a, b, c = &v, a, b
        } else if *a > v && (b == nil || v > *b) {
            b, c = &v, b
        } else if b != nil && v < *b && (c == nil || v > *c) {
            c = &v
        }
    }

    if c == nil {
        return *a
    }
    return *c
}
```
