---
layout: post
title: "Algorithm Backtracking"
date: 2021-01-02 09:00:00 +0800
tags: Algorithm Leetcode
---

记录 Backtracking 的算法实现

### "78. Subsets" Golang

```Golang
func subsets(nums []int) (ret [][]int) {
    n := len(nums)
    set := []int{}

    var dfs func(cur int)
    dfs = func(cur int){
        if cur == n {
            ret = append(ret, append([]int{}, set...))
            return
        }
        dfs(cur + 1)
        set = append(set, nums[cur])
        dfs(cur + 1)
        set = set[:len(set)-1]
    }
    dfs(0)
    return
}
```
