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
