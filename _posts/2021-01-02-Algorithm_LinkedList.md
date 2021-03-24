---
layout: post
title: "Algorithm Linked List"
date: 2021-01-02 09:00:00 +0800
tags: Algorithm Leetcode
---

记录 Linked List 的算法实现

### "82. Remove Duplicates from Sorted List II"

```Go
func deleteDuplicates(head *ListNode) *ListNode {
    dummyHead := &ListNode{ Val: -101, Next: head }
    for preTail, cur := dummyHead, head; cur != nil; {
        if cur.Next != nil && cur.Val == cur.Next.Val {
            curVal := cur.Val
            for cur != nil && cur.Val == curVal {
                cur = cur.Next
            }
            preTail.Next = cur
        } else {
            preTail = cur
            cur = cur.Next
        }
    }
    return dummyHead.Next
}
```

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

### "239. Sliding Window Maximum" Golang

```Go
func maxSlidingWindow(nums []int, k int) []int {
    l := list.New()

    push := func(i int) {
        for l.Len() > 0 && nums[l.Back().Value.(int)] <= nums[i] {
            l.Remove(l.Back())
        }
        l.PushBack(i)
    }

    pop := func(i int) {
        for l.Len() > 0 && l.Front().Value.(int) <= i-k {
            l.Remove(l.Front())
        }
    }

    for i := 0; i < k; i++ {
        push(i)
    }
    ret := make([]int, 1, len(nums)-k+1)
    ret[0] = nums[l.Front().Value.(int)]

    for i := k; i < len(nums); i++ {
        push(i)
        pop(i)
        ret = append(ret, nums[l.Front().Value.(int)])
    }

    return ret
}
```
