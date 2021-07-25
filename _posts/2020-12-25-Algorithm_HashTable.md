---
layout: post
title: "Algorithm HashTable"
date: 2020-12-25 22:00:00 +0800
tags: Algorithm Leetcode
---

记录 HashTable 的算法实现

### "49. Group Anagrams" Golang

- 在 Go 中，map 的 key 可以是较复杂类型，只要符合`Comparable`特性即可，这里就应用数组类型作为 key

```Go
func groupAnagrams(strs []string) [][]string {
    m := make(map[[26]int][]string)

    for _, s := range strs {
        id := [26]int{}
        for _, c := range s {
            id[c-'a']++
        }
        m[id] = append(m[id], s)
    }

    ret := make([][]string, 0, len(m))
    for _, v := range m {
        ret = append(ret, v)
    }

    return ret
}
```

### "560. Subarray Sum Equals K" Golang

```Go
func subarraySum(nums []int, k int) (ret int) {
    m, sum := map[int]int{0:1}, 0

    for _, n := range nums {
        sum += n
        if count, exists := m[sum-k]; exists {
            ret += count
        }
        m[sum]++
    }

    return ret
}
```

### "930. Binary Subarrays With Sum"

```Go
func numSubarraysWithSum(nums []int, goal int) (ret int) {
    n := len(nums)
    preSum := make([]int, n+1)
    for i, sum := 0, 0; i < n; i++ {
        sum += nums[i]
        preSum[i+1] = sum
    }
    m := make(map[int]int)
    for _, v := range preSum {
        ret += m[v-goal]
        m[v]++
    }
    return ret
}
```

### "974. Subarray Sums Divisible by K"

```Go
func subarraysDivByK(A []int, K int) int {
    mem, sum, count := map[int]int{0:1}, 0, 0
    for _, a := range A {
        sum += a
        module := (sum%K+K)%K
        count += mem[module]
        mem[module]++
    }
    return count
}
```

### "1743. Restore the Array From Adjacent Pairs"

```Go
func restoreArray(adjacentPairs [][]int) (ret []int) {
    n := len(adjacentPairs)
    g := make(map[int][]int, n+1)
    for _, p := range adjacentPairs {
        g[p[0]] = append(g[p[0]], p[1])
        g[p[1]] = append(g[p[1]], p[0])
    }

    for k, e := range g {
        if len(e) == 1 {
            ret = append(ret, k, e[0])
            break
        }
    }

    for i := 1; i < n; i++ {
        if g[ret[i]][0] == ret[i-1] {
            ret = append(ret, g[ret[i]][1])
        } else {
            ret = append(ret, g[ret[i]][0])
        }
    }

    return ret
}
```
