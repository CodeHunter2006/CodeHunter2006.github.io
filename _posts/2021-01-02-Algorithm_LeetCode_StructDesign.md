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
import "math/rand"

type RandomizedCollection struct {
    m map[int]map[int]struct{}
    b []int
}

func Constructor() RandomizedCollection {
    return RandomizedCollection{
        m : make(map[int]map[int]struct{}),
    }
}

func (this *RandomizedCollection) Insert(val int) bool {
    _, ok := this.m[val]
    if !ok {
        this.m[val] = make(map[int]struct{})
    }
    this.m[val][len(this.b)] = struct{}{}
    this.b = append(this.b, val)
    return !ok
}

func (this *RandomizedCollection) Remove(val int) bool {
    set, ok := this.m[val]
    if !ok {
        return false
    }
    var tarDelIdx int
    for k := range set {
        tarDelIdx = k
        break
    }
    lastVal := this.b[len(this.b)-1]

    delete(this.m[lastVal], len(this.b)-1)
    delete(set, tarDelIdx)
    this.b = this.b[:len(this.b)-1]

    if tarDelIdx != len(this.b) {
        this.m[lastVal][tarDelIdx] = struct{}{}
        this.b[tarDelIdx] = lastVal
    }

    if len(set) == 0 {
        delete(this.m, val)
    }
    return true
}

func (this *RandomizedCollection) GetRandom() int {
    return this.b[rand.Intn(len(this.b))]
}
```

```Go
type VFE struct {
    v int   // value
    f int   // frequency
    e *list.Element // point in list in one frequency
}

type LFUCache struct {
    capacity int
    minFreq int
    k2vfe map[int]*VFE
    f2l map[int]*list.List
}

func Constructor(capacity int) LFUCache {
    res := LFUCache {
        capacity : capacity,
        minFreq : 0,
        k2vfe : make(map[int]*VFE),
        f2l : make(map[int]*list.List),
    }
    return res
}

func (this *LFUCache) add2List(freq, key int) *list.Element {
    if _, exists := this.f2l[freq]; !exists {
        this.f2l[freq] = list.New()
    }

    return this.f2l[freq].PushBack(key)
}

func (this *LFUCache) Get(key int) int {
    if vfe, exist := this.k2vfe[key]; exist {
        this.f2l[vfe.f].Remove(vfe.e)
        vfe.f++
        vfe.e = this.add2List(vfe.f, key)

        if this.f2l[this.minFreq].Len() == 0 {
            this.minFreq++
        }
        return vfe.v
    }
    return -1
}

func (this *LFUCache) Put(key int, value int)  {
    if this.capacity == 0 {
        return
    }
    if vfe, exist := this.k2vfe[key]; exist {
        this.f2l[vfe.f].Remove(vfe.e)
        if vfe.f == this.minFreq && this.f2l[vfe.f].Len() == 0 {
            this.minFreq++
        }
        vfe.f++
        vfe.v = value
        vfe.e = this.add2List(vfe.f, key)
        return
    }

    if len(this.k2vfe) >= this.capacity {
        rmKey := this.f2l[this.minFreq].Front().Value.(int)
        delete(this.k2vfe, rmKey)
        this.f2l[this.minFreq].Remove(this.f2l[this.minFreq].Front())
    }

    this.k2vfe[key] = &VFE {
        v : value,
        f : 1,
        e : this.add2List(1, key),
    }
    this.minFreq = 1
}
```
