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

### "380. Insert Delete GetRandom O(1)"

```Go
type RandomizedSet struct {
    m map[int]int
    b []int
}

func Constructor() RandomizedSet {
    return RandomizedSet{
        m : make(map[int]int),
    }
}

func (this *RandomizedSet) Insert(val int) bool {
    if _, ok := this.m[val]; ok {
        return false
    }
    this.m[val] = len(this.b)
    this.b = append(this.b, val)
    return true
}

func (this *RandomizedSet) Remove(val int) bool {
    idx, ok := this.m[val]
    if !ok {
        return false
    }

    lastVal := this.b[len(this.b)-1]
    this.m[lastVal] = idx
    this.b[idx] = lastVal

    delete(this.m, val)
    this.b = this.b[:len(this.b)-1]
    return true
}

func (this *RandomizedSet) GetRandom() int {
    return this.b[rand.Intn(len(this.b))]
}
```

### "381. Insert Delete GetRandom O(1) - Duplicates allowed"

```Go

```
