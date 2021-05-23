---
layout: post
title: "Leetcode Sliding Window"
date: 2020-07-05 21:00:00 +0800
tags: Algorithm Leetcode
---

## 剑指 Offer 59 - I. 滑动窗口的最大值

[题目](https://leetcode-cn.com/problems/hua-dong-chuang-kou-de-zui-da-zhi-lcof/)

- 解法
  在滑窗基础上利用双向队列维护最大值备选有序队列
  dq 可以采取不同数据结构，如果想节省内存，可以用`container/list`
  本题对内存不敏感，所以提前声明一个较长的 slice

```Go
func maxSlidingWindow(nums []int, k int) []int {
    if len(nums) < k {
        return nil
    }

    res := make([]int, 0, len(nums) - k + 1)
    dq := make([]int, 0, len(nums))

    for i, num := range nums {
        // 最大值弹出
        if i >= k && len(dq) > 0 && dq[0] == nums[i - k] {
            dq = dq[1:]
        }

        // 加入备选有序队列
        for len(dq) > 0 && dq[len(dq)-1] < num {
            dq = dq[:len(dq)-1]
        }
        dq = append(dq, num)

        if i + 1 >= k {
            res = append(res, dq[0])
        }
    }

    return res
}
```

### "1438. Longest Continuous Subarray With Absolute Diff Less Than or Equal to Limit" SlidingWindow+MonotoneQueue Golang

```Go
func longestSubarray(nums []int, limit int) (ret int) {
    left := 0
    var minQue, maxQue []int
    for right, v := range nums {
        for len(minQue) > 0 && minQue[len(minQue)-1] > v {
            minQue = minQue[:len(minQue)-1]
        }
        minQue = append(minQue, v)
        for len(maxQue) > 0 && maxQue[len(maxQue)-1] < v {
            maxQue = maxQue[:len(maxQue)-1]
        }
        maxQue = append(maxQue, v)
        for len(maxQue) > 0 && len(minQue) > 0 &&
        (maxQue[0] - minQue[0] > limit) {
            if nums[left] == maxQue[0] {
                maxQue = maxQue[1:]
            }
            if nums[left] == minQue[0] {
                minQue = minQue[1:]
            }
            left++
        }
        ret = max(ret, right-left+1)
    }
    return ret
}

func max(a, b int) int {
    if a > b {
        return a
    }
    return b
}
```

### "395. Longest Substring with At Least K Repeating Characters" Golang

```Go
func longestSubstring(s string, k int) (ret int) {
    for n := 1; n <= 26; n++ {
        // 窗口内字母种类、窗口内小于k个的字母种类、左边
        var setSize, lessK, l int
        var record [26]int  // 窗内字母数量统计
        for r, c := range s {
            index := c-'a'
            if record[index] == 0 {
                setSize++
                lessK++
            }
            record[index]++
            if record[index] == k {
                lessK--
            }

            for setSize > n {
                idx := s[l]-'a'
                if record[idx] == k {
                    lessK++
                }
                record[idx]--
                if record[idx] == 0 {
                    setSize--
                    lessK--
                }
                l++
            }

            if lessK == 0 {
                ret = max(ret, r-l+1)
            }
        }
    }
    return ret
}
```

### "1044. Longest Duplicate Substring" RollingHash

```Go
func longestDupSubstring(s string) (ret string) {
    find := func(k int) string {
        if k == 0 {return ""}
        m := make(map[int64][]int, len(s)-k+1)  // map[hashCode][]index
        var hashCode int64
        pow := Power(k)
        for i := range s {
            hashCode *= MOD
            if i >= k {
                hashCode -= int64(s[i-k]) * pow
            }
            hashCode += int64(s[i])
            if i < k-1 {
                continue
            }
            if sli, ok := m[hashCode]; ok {
                for _, idx := range sli {
                    if string(s[i-k+1:i+1]) == string(s[idx: idx+k]) {
                        return string(s[i-k+1:i+1])
                    }
                }
            }

            m[hashCode] = append(m[hashCode], i-k+1)
        }
        return ""
    }

    l, r := 0, len(s)
    for l+1 < r {
        mid := (l+r)>>1
        if ret = find(mid); ret != "" {
            l = mid
        } else {
            r = mid
        }
    }

    return find(l)
}


const MOD = 26
func Power(y int) (ret int64) {
    for ret = 1; y > 0; y-- {
        ret *= MOD
    }
    return ret
}
```

### "1316. Distinct Echo Substrings"

### "1392. Longest Happy Prefix"

```Go
func longestPrefix(s string) (ret string) {
    const MOD = 1e9+7
    const BASE = 26
    last := len(s)-1

    var preHash, postHash, pow int64 = 0, 0, 1
    for i := 0; i < last; i++ {
        preHash = (preHash*BASE + int64(s[i]-'a'))%MOD
        postHash = (postHash + int64(s[last-i]-'a') * pow)%MOD
        pow = (pow*BASE)%MOD
        if preHash == postHash {
            ret = s[:i+1]
        }
    }

    return ret
}
```
