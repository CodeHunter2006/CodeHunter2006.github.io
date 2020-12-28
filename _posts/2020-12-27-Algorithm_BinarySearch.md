---
layout: post
title: "Algorithm Binary Search"
date: 2020-12-27 09:00:00 +0800
tags: Algorithm Leetcode
---

记录 Binary Search 的算法实现

### "33. Search in Rotated Sorted Array" Golang

```Go
func search(nums []int, target int) (ret int) {
    if len(nums) == 0 {
        return -1
    }
    //defer func(){ fmt.Println(nums[0], nums[len(nums)-1], ret) }()
    mid := len(nums)>>1

    if nums[0] <= nums[len(nums)-1] {
        if target > nums[len(nums)-1] || target < nums[0] {
            return -1
        } else if target == nums[mid] {
            return mid
        }
    }

    if ret = search(nums[:mid], target); ret != -1 {
        return ret
    } else if ret = search(nums[mid:], target); ret != -1 {
        return mid + ret
    }

    return -1
}
```

### "34. Find First and Last Position of Element in Sorted Array" Golang

- 可以利用`sort.SearchInts(a []int, x int) int`，用二分查找找到元素第一次出现的下标，如果没有找到则返回元素插入操作的下标

```Go
func searchRange(nums []int, target int) []int {
    l := sort.SearchInts(nums, target)
    if l == len(nums) || nums[l] != target {
        return []int{-1, -1}
    }
    r := sort.SearchInts(nums, target + 1) - 1
    return []int{l, r}
}
```
