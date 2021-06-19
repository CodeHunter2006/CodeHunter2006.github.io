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

### "1239. Maximum Length of a Concatenated String with Unique Characters"

```Go
func maxLength(arr []string) int {
    bitArr := make([]uint32, 0, len(arr))
    bitCount := make([]int, 0, len(arr))

outer:
    for _, s := range arr {
        bit, count := uint32(0), 0
        for _, c := range s {
            if tmp := uint32(1<<(c-'a')); bit & tmp > 0 {
                continue outer
            } else {
                bit |= tmp
                count++
            }
        }
        bitArr = append(bitArr, bit)
        bitCount = append(bitCount, count)
    }
    ret := 0
    dfs(bitArr, bitCount, &ret, 0, 0, 0)
    return ret
}

func dfs (bitArr []uint32, bitCount[]int, maxInt *int, pos int, bitSum uint32, count int) {
    if pos >= len(bitArr) { return }
    if (bitArr[pos] & bitSum) == 0 {
        bitSum |= bitArr[pos]
        count += bitCount[pos]
        if count > *maxInt {
            *maxInt = count
        }
        dfs(bitArr, bitCount, maxInt, pos+1, bitSum, count)
        bitSum &^= bitArr[pos]
        count -= bitCount[pos]
    }
    dfs(bitArr, bitCount, maxInt, pos+1, bitSum, count)
}
```
