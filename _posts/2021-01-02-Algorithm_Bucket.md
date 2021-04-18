---
layout: post
title: "Algorithm OrderedMap"
date: 2021-01-02 09:00:00 +0800
tags: Algorithm Leetcode
---

记录 Bucket 的算法实现

### "220. Contains Duplicate III"

```Go
func containsNearbyAlmostDuplicate(nums []int, k int, t int) bool {
    bucket := make(map[int]int)
    for i, v := range nums {
        id := getID(v, t+1)
        if y, ok := bucket[id]; ok {
            fmt.Println(1, id, v, y)
            return true
        } else if y, ok := bucket[id-1]; ok && abs(v-y) <= t {
            fmt.Println(2, id, v, y)
            return true
        } else if y, ok := bucket[id+1]; ok && abs(v-y) <= t {
            fmt.Println(3, id, v, y)
            return true
        }
        bucket[id] = v
        if i >= k {
            delete(bucket, getID(nums[i-k], t+1))
        }
    }
    return false
}

func getID(x, w int) int {
    if x >= 0 {
        return x/w
    }
    return (x+1)/w -1
}

func abs(x int) int {
    if x >= 0 {
        return x
    }
    return -x
}
```

### "740. Delete and Earn"

```Go
func deleteAndEarn(nums []int) int {
    var bucket [1e4+1]int
    for _, n := range nums {
        bucket[n] += n
    }
    first, second := bucket[0], max(bucket[0], bucket[1])
    for i := 2; i < len(bucket); i++ {
        first, second = second, max(first+bucket[i], max(first, second))
    }
    return max(first, second)
}
```
