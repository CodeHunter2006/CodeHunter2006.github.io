---
layout: post
title: "Algorithm Backtracking"
date: 2021-01-02 09:00:00 +0800
tags: Algorithm Leetcode
---

记录 Backtracking 的算法实现

### "39. Combination Sum"

```Go
func combinationSum(candidates []int, target int) (ret [][]int) {
    var pre []int
    var dfs func([]int, int)
    dfs = func(candidates []int, tar int) {
        if tar == 0 {
            ret = append(ret, append([]int{}, pre...))
            return
        }
        for i, v := range candidates {
            if v > tar { continue }
            pre = append(pre, v)
            dfs(candidates[i:], tar-v)
            pre = pre[:len(pre)-1]
        }
    }
    dfs(candidates, target)
    return ret
}
```

### "40. Combination Sum II"

```Go
func combinationSum2(candidates []int, target int) (ret [][]int) {
    sort.Ints(candidates)
    var dfs func([]int, []int, int)
    dfs = func(candidates, pre []int, tar int) {
        if tar == 0 {
            ret = append(ret, append([]int{}, pre...))
            return
        }
        for i := 0; i < len(candidates); i++ {
            if i > 0 && candidates[i] == candidates[i-1] {
                continue
            }
            if next := tar - candidates[i]; next >= 0 {
                tmp := append(pre, candidates[i])
                dfs(candidates[i+1:], tmp, next)
            } else {
                break
            }
        }
    }
    dfs(candidates, nil, target)
    return ret
}
```

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

### "216. Combination Sum III"

```Go
func combinationSum3(k int, n int) (ret [][]int) {
    var bigArr []int
    for i := 1; i <= 9 && i <= n; i++ {
        bigArr = append(bigArr, i)
    }

    var pre []int
    var dfs func([]int, int, int)
    dfs = func(arr []int, tar, k int) {
        if tar == 0 && k == 0 {
            ret = append(ret, append([]int{}, pre...))
            return
        } else if k == 0 {
            return
        }
        for i, v := range arr {
            if v > tar {
                break
            }
            pre = append(pre, v)
            dfs(arr[i+1:], tar-v, k-1)
            pre = pre[:len(pre)-1]
        }
    }
    dfs(bigArr, n, k)
    return ret
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
