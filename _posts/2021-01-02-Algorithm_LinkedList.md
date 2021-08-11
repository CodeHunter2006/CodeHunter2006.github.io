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

### "138. Copy List with Random Pointer"

```Go
// hash map
var gMap = make(map[*Node]*Node)
func copyRandomList(head *Node) *Node {
    if head == nil {
        return nil
    } else if gMap[head] != nil {
        return gMap[head]
    }
    newNode := &Node{ Val: head.Val }
    gMap[head] = newNode
    newNode.Next = copyRandomList(head.Next)
    newNode.Random = copyRandomList(head.Random)
    return newNode
}
```

```Go
// link node
func copyRandomList(head *Node) *Node {
    if head == nil {
        return nil
    }
    for cur := head; cur != nil; cur = cur.Next.Next {
        cur.Next = &Node{ Val: cur.Val, Next: cur.Next }
    }
    for cur := head; cur != nil; cur = cur.Next.Next {
        if cur.Random != nil {
            cur.Next.Random = cur.Random.Next
        }
    }
    newHead := &Node{}
    for cur, pre := head, newHead; cur != nil; cur, pre = cur.Next, pre.Next {
        pre.Next, cur.Next = cur.Next, cur.Next.Next
    }
    return newHead.Next
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

### "148. Sort List"

```Go
func sortList(head *ListNode) *ListNode {
    return sort(head, nil)
}

func sort(head, tail *ListNode) *ListNode {
    if head == nil {
        return head
    } else if head.Next == tail {
        head.Next = nil
        return head
    }
    slow, fast := head, head.Next
    for fast != tail {
        slow, fast = slow.Next, fast.Next
        if fast != tail {
            fast = fast.Next
        }
    }

    return merge(sort(head, slow), sort(slow, tail))
}

func merge(h1, h2 *ListNode) *ListNode {
    dummyHead := &ListNode{}
    pre := dummyHead
    for h1 != nil && h2 != nil {
        if h1.Val < h2.Val {
            pre.Next, h1 = h1, h1.Next
        } else {
            pre.Next, h2 = h2, h2.Next
        }
        pre = pre.Next
    }
    if h1 != nil {
        pre.Next = h1
    } else if h2 != nil {
        pre.Next = h2
    }
    return dummyHead.Next
}
```

### "160. Intersection of Two Linked Lists"

```Go
/**
 * Definition for singly-linked list.
 * type ListNode struct {
 *     Val int
 *     Next *ListNode
 * }
 */
func getIntersectionNode(headA, headB *ListNode) *ListNode {
    p1, p2 := headA, headB
    for p1 != p2 {
        if p1 == nil {
            p1 = headB
        } else {
            p1 = p1.Next
        }
        if p2 == nil {
            p2 = headA
        } else {
            p2 = p2.Next
        }
    }
    return p1
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
