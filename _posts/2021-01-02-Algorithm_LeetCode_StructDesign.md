---
layout: post
title: "Algorithm Struct Design"
date: 2021-01-02 09:00:00 +0800
tags: Algorithm Leetcode
---

记录 LeetCode 中常见的据结构设计

### "146. LRU Cache" Golang

```Go
import "container/list"

type LRUCache struct {
    *list.List
    m map[int]*list.Element
    size int
}

type kv struct {
    Key int
    Value int
}

func Constructor(capacity int) LRUCache {
    return LRUCache{
        List: list.New(),
        m : make(map[int]*list.Element),
        size : capacity,
    }
}

func (this *LRUCache) Get(key int) int {
    if element, exists := this.m[key]; exists {
        this.MoveToBack(element)
        return element.Value.(*kv).Value
    }
    return -1
}


func (this *LRUCache) Put(key int, value int)  {
    tmpKV := &kv{Key : key, Value : value}
    if element, exists := this.m[key]; exists {
        element.Value = tmpKV
        this.MoveToBack(element)
        return
    }

    element := this.PushBack(tmpKV)
    this.m[key] = element

    for len(this.m) > this.size {
        delete(this.m, this.Front().Value.(*kv).Key)
        this.Remove(this.Front())
    }
}
```
