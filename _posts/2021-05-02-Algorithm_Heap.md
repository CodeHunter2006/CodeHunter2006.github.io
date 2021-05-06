---
layout: post
title: "Algorithm Heap"
date: 2021-05-02 09:00:00 +0800
tags: Algorithm Leetcode
---

记录 Heap 的算法实现

### "703. Kth Largest Element in a Stream"

```Go
type IntHeap []int

func (h IntHeap) Len() int {
    return len(h)
}

func (h IntHeap) Less(a, b int) bool {
    return h[a] < h[b]
}

func (h IntHeap) Swap(a, b int) {
    h[a], h[b] = h[b], h[a]
}

func (h *IntHeap) Push(x interface{}) {
    *h = append(*h, x.(int))
}

func (h *IntHeap) Pop() interface{} {
    res := (*h)[len(*h)-1]
    *h = (*h)[:len(*h)-1]
    return res
}

func (h IntHeap) Top() int {
    return h[0]
}

type KthLargest struct {
    size int
    intHeap *IntHeap
}
func Constructor(k int, nums []int) KthLargest {
    res := KthLargest{
        size : k,
        intHeap : (*IntHeap)(&nums),
    }
    heap.Init(res.intHeap)
    return res
}

func (this *KthLargest) Add(val int) int {
    heap.Push(this.intHeap, val)
    for this.intHeap.Len() > this.size {
        heap.Pop(this.intHeap)
    }
    return this.intHeap.Top()
}
```

### "295. Find Median from Data Stream"

```Go
type MyHeap struct {
    array []int
    isMax bool
}

func NewHeap(isMax bool) *MyHeap {
    return &MyHeap{
        array : []int{},
        isMax : isMax,
    }
}

func (h *MyHeap) Len() int {
    return len(h.array)
}

func (h *MyHeap) Less(a, b int) bool {
    if h.isMax {
        return h.array[a] > h.array[b]
    }
    return h.array[a] < h.array[b]
}

func (h *MyHeap) Swap(a, b int) {
    h.array[a], h.array[b] = h.array[b], h.array[a]
}

func (h *MyHeap) Push(x interface{}) {
    h.array = append(h.array, x.(int))
}

func (h *MyHeap) Pop() (x interface{}) {
    x = h.array[len(h.array)-1]
    h.array = h.array[0 : len(h.array)-1]
    return
}

func (h *MyHeap) Top() int {
    return h.array[0]
}

type MedianFinder struct {
    minHeap *MyHeap
    maxHeap *MyHeap
}

/** initialize your data structure here. */
func Constructor() MedianFinder {
    return MedianFinder{
        minHeap : NewHeap(false),
        maxHeap : NewHeap(true),
    }
}

func (this *MedianFinder) AddNum(num int)  {
    heap.Push(this.minHeap, num)
    // 注意这里要做一下流转，避免新元素造成大小顺序错误
    if this.maxHeap.Len() > 0 {
        heap.Push(this.minHeap, heap.Pop(this.maxHeap))
    }

    for this.minHeap.Len() > this.maxHeap.Len() {
        heap.Push(this.maxHeap, heap.Pop(this.minHeap))
    }
}

func (this *MedianFinder) FindMedian() float64 {
    if (this.minHeap.Len() + this.maxHeap.Len()) & 1 == 1 {
        return float64(this.maxHeap.Top())
    }
    return float64(this.minHeap.Top() + this.maxHeap.Top()) / 2.0
}
```
