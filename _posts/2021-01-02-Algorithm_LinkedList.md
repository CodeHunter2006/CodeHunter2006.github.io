---
layout: post
title: "Algorithm Linked List"
date: 2021-01-02 09:00:00 +0800
tags: Algorithm Leetcode
---

记录 Linked List 的算法实现

### "142. Linked List Cycle II" Golang

```Go
func detectCycle(head *ListNode) *ListNode {
    fast, slow := head, head
    for fast != nil {
        if fast.Next == nil {
            return nil
        }
        slow, fast = slow.Next, fast.Next.Next

        if slow == fast {
            break
        }
    }

    if fast == nil {
        return nil
    }

    for fast = head; fast != slow; fast, slow = fast.Next, slow.Next{}

    return fast
}
```
