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
