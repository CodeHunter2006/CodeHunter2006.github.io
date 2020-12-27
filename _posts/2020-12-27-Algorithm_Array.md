---
layout: post
title: "Algorithm Array"
date: 2020-12-27 09:00:00 +0800
tags: Algorithm Leetcode
---

记录 Array 的算法实现

### "31. Next Permutation" Golang

```Go
func nextPermutation(nums []int)  {
    if len(nums) < 2 {
        return
    }

    l := len(nums) - 1
    for ; l > 0 && nums[l-1] >= nums[l]; l-- {
    }

    if l > 0 {
        r := l
        for i := l; i < len(nums) && nums[i] > nums[l-1]; r, i = i, i+1 {
        }
        nums[l-1], nums[r] = nums[r], nums[l-1]
    }

    for i, j := l, len(nums)-1; i < j; i, j = i+1, j-1 {
        nums[i], nums[j] = nums[j], nums[i]
    }
}
```
