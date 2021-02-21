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
