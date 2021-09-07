---
layout: post
title: "Algorithm Struct Design"
date: 2021-01-02 09:00:00 +0800
tags: Algorithm Leetcode
---

记录 LeetCode 中常见的据结构设计

### "65. Valid Number"

```Go
type State int
type CharType int

const (
    STATE_INITIAL State = iota
    STATE_INT_SIGN
    STATE_INTEGER
    STATE_POINT
    STATE_POINT_WITHOUT_INT
    STATE_FRACTION
    STATE_EXP
    STATE_EXP_SIGN
    STATE_EXP_NUMBER
    STATE_END
)

const (
    CHAR_NUMBER CharType = iota
    CHAR_EXP
    CHAR_POINT
    CHAR_SIGN
    CHAR_ILLEGAL
)

func toCharType(ch byte) CharType {
    switch ch {
    case '0', '1', '2', '3', '4', '5', '6', '7', '8', '9':
        return CHAR_NUMBER
    case 'e', 'E':
        return CHAR_EXP
    case '.':
        return CHAR_POINT
    case '+', '-':
        return CHAR_SIGN
    default:
        return CHAR_ILLEGAL
    }
}

func isNumber(s string) bool {
    transfer := map[State]map[CharType]State{
        STATE_INITIAL: map[CharType]State{
            CHAR_NUMBER: STATE_INTEGER,
            CHAR_POINT:  STATE_POINT_WITHOUT_INT,
            CHAR_SIGN:   STATE_INT_SIGN,
        },
        STATE_INT_SIGN: map[CharType]State{
            CHAR_NUMBER: STATE_INTEGER,
            CHAR_POINT:  STATE_POINT_WITHOUT_INT,
        },
        STATE_INTEGER: map[CharType]State{
            CHAR_NUMBER: STATE_INTEGER,
            CHAR_EXP:    STATE_EXP,
            CHAR_POINT:  STATE_POINT,
        },
        STATE_POINT: map[CharType]State{
            CHAR_NUMBER: STATE_FRACTION,
            CHAR_EXP:    STATE_EXP,
        },
        STATE_POINT_WITHOUT_INT: map[CharType]State{
            CHAR_NUMBER: STATE_FRACTION,
        },
        STATE_FRACTION: map[CharType]State{
            CHAR_NUMBER: STATE_FRACTION,
            CHAR_EXP:    STATE_EXP,
        },
        STATE_EXP: map[CharType]State{
            CHAR_NUMBER: STATE_EXP_NUMBER,
            CHAR_SIGN:   STATE_EXP_SIGN,
        },
        STATE_EXP_SIGN: map[CharType]State{
            CHAR_NUMBER: STATE_EXP_NUMBER,
        },
        STATE_EXP_NUMBER: map[CharType]State{
            CHAR_NUMBER: STATE_EXP_NUMBER,
        },
    }
    state := STATE_INITIAL
    for i := 0; i < len(s); i++ {
        typ := toCharType(s[i])
        if _, ok := transfer[state][typ]; !ok {
            return false
        } else {
            state = transfer[state][typ]
        }
    }
    return state == STATE_INTEGER || state == STATE_POINT || state == STATE_FRACTION || state == STATE_EXP_NUMBER || state == STATE_END
}
```

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

### "295. Find Median from Data Stream"

```Go
type minHeap struct {
    sort.IntSlice
}
func (p *minHeap) Push(x interface{}) {
    p.IntSlice = append(p.IntSlice, x.(int))
}
func (p *minHeap) Pop() interface{} {
    n := len(p.IntSlice)
    ret := p.IntSlice[n-1]
    p.IntSlice = p.IntSlice[:n-1]
    return ret
}

type maxHeap struct {
    minHeap
}
func (p *maxHeap) Less(a, b int) bool {
    // 这里如果用 !p.Less(a, b) 会造成死循环递归
    return !p.minHeap.Less(a, b)
}

type MedianFinder struct {
    minH *minHeap
    maxH *maxHeap
}
func Constructor() MedianFinder {
    return MedianFinder{
        minH: &minHeap{},
        maxH: &maxHeap{},
    }
}
func (this *MedianFinder) AddNum(num int)  {
    // 首先交换一次元素以确保大小值合理，即 minH 放较大值、maxH 放较小值
    heap.Push(this.maxH, num)
    heap.Push(this.minH, heap.Pop(this.maxH))
    if this.minH.Len() > this.maxH.Len()+1 {
        heap.Push(this.maxH, heap.Pop(this.minH))
    }
}
func (this *MedianFinder) FindMedian() (ret float64) {
    if (this.minH.Len()+this.maxH.Len()) & 1 == 0 {
        return float64(this.minH.IntSlice[0]+this.maxH.IntSlice[0])/2
    }
    return float64(this.minH.IntSlice[0])
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

### "460. LFU Cache"

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

### "480. Sliding Window Median"

```Go
type hp struct {
    sort.IntSlice
    size int    // 有效元素数量
    delayDelMap map[int]int
    isBigHeap bool  // true 表示最大堆，保存负数；默认 false 表示最小堆
}
func (p *hp) Push(x interface{}) {
    p.IntSlice = append(p.IntSlice, x.(int))
}
func (p *hp) Pop() interface{} {
    ret := p.IntSlice[p.Len()-1]
    p.IntSlice = p.IntSlice[:p.Len()-1]
    return ret
}
func (p *hp) push(x int) {
    p.size++
    heap.Push(p, x)
}
func (p *hp) pop() int {
    p.size--
    return heap.Pop(p).(int)
}
func (p *hp) prune() {
    // prune 时 size 不变
    for p.Len() > 0 {
        num := p.IntSlice[0]
        if p.isBigHeap {
            num = -num
        }
        if d, has := p.delayDelMap[num]; has {
            if d > 1 {
                p.delayDelMap[num]--
            } else {
                delete(p.delayDelMap, num)
            }
            heap.Pop(p)
        } else {
            break
        }
    }
}

type slidingWindow struct {
    smallHeap *hp   // 较小元素，是最大堆，放负数
    largeHeap *hp
    delayDelMap map[int]int
    windowSize int
}
func NewWindow(size int) *slidingWindow {
    m := make(map[int]int)
    return &slidingWindow{
        smallHeap: &hp{delayDelMap: m, isBigHeap: true},
        largeHeap: &hp{delayDelMap: m},
        delayDelMap: m,
        windowSize: size,
    }
}
func (p *slidingWindow) makeBalance() {
    if p.smallHeap.size > p.largeHeap.size+1 {  // 较小数堆元素允许偏多1
        p.largeHeap.push(-p.smallHeap.pop())
        p.smallHeap.prune()
    } else if p.largeHeap.size > p.smallHeap.size {
        p.smallHeap.push(-p.largeHeap.pop())
        p.largeHeap.prune()
    }
}
func (p *slidingWindow) insert(num int) {
    if p.smallHeap.Len() == 0 || num <= -p.smallHeap.IntSlice[0] {
        p.smallHeap.push(-num)
    } else {
        p.largeHeap.push(num)
    }
    p.makeBalance()
}
func (p *slidingWindow) erase(num int) {
    p.delayDelMap[num]++
    if num <= -p.smallHeap.IntSlice[0] {
        p.smallHeap.size--
        p.smallHeap.prune()
    } else {
        p.largeHeap.size--
        p.largeHeap.prune()
    }
    p.makeBalance()
}
func (p *slidingWindow) getMedian() float64 {
    if p.windowSize & 1 > 0 {
        return float64(-p.smallHeap.IntSlice[0])
    } else {
        return float64(-p.smallHeap.IntSlice[0]+p.largeHeap.IntSlice[0])/2
    }
}

func medianSlidingWindow(nums []int, k int) []float64 {
    n := len(nums)
    sw := NewWindow(k)
    for _, v := range nums[:k] {
        sw.insert(v)
    }
    ret := make([]float64, 0, n-k+1)
    ret = append(ret, sw.getMedian())

    for i := k; i < n; i++ {
        sw.insert(nums[i])
        sw.erase(nums[i-k])
        //fmt.Println("insert",nums[i],"erase",nums[i-k],"ret", sw.getMedian(),sw.delayDelMap,sw.smallHeap.IntSlice,sw.largeHeap.IntSlice)
        ret = append(ret, sw.getMedian())
    }

    return ret
}
```
