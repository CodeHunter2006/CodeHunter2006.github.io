---
layout: post
title: "Algorithm Stack"
date: 2020-12-27 09:00:00 +0800
tags: Algorithm Leetcode
---

记录 Stack 的算法实现

### "32. Longest Valid Parentheses" Golang

```Go
func longestValidParentheses(s string) int {
    stack := make([]int, 0, len(s))
    stack = append(stack, -1)
    maxLen := 0

    updateMax := func(val int) {
        if val > maxLen {
            maxLen = val
        }
    }

    for i := 0; i < len(s); i++ {
        if s[i] == ')' {
            if stack[len(stack)-1] != -1 && s[stack[len(stack)-1]] == '(' {
                stack = stack[:len(stack)-1]
                updateMax(i - stack[len(stack)-1])
            } else {
                stack = append(stack, i)
            }
        } else {
            stack = append(stack, i)
        }
    }

    return maxLen
}
```

### "84. Largest Rectangle in Histogram" Golang

```Go
func largestRectangleArea(heights []int) (ret int) {
    stack := []int{-1}

    max := func(val int) {
        if val > ret {
            ret = val
        }
    }

    for i := 0; i < len(heights); i++ {
        for stack[len(stack)-1] != -1 && heights[stack[len(stack)-1]] >= heights[i] {
            top := stack[len(stack)-1]
            stack = stack[:len(stack)-1]
            max(heights[top] * (i - 1 - stack[len(stack)-1]) )
        }
        stack = append(stack, i)
    }

    for stack[len(stack)-1] != -1 {
        top := stack[len(stack)-1]
        stack = stack[:len(stack)-1]
        max(heights[top] * (len(heights) - 1 - stack[len(stack)-1]))
    }

    return ret
}
```

### "1063. Number of Valid Subarrays"

```Go
func validSubarrays(nums []int) (ret int) {
    nums = append(nums, -1)
    var stack []int
    for i, v := range nums {
        for len(stack) > 0 && nums[stack[len(stack)-1]] > v {
            ret += i - stack[len(stack)-1]
            stack = stack[:len(stack)-1]
        }
        stack = append(stack, i)
    }
    return ret
}
```
