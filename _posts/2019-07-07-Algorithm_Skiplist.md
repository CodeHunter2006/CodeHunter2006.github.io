---
layout: post
title:  "算法学习，极限数据结构之——跳跃表(Skip List)"
date:   2019-07-07 10:00:00 +0800
tags: Algorithm
---
# 数据结构特性
![Skip List](/assets/images/2019-07-07-Algorithm_Skiplist_1.jpg)
跳跃表，Skip List<br/>
* 由多层结构组成，每一层是一个有序的链表
* 最底层的链表包含所有元素
* 每个节点包含两个指针，一个下一个元素、一个下一层元素，如果在上层链表出现了那在下层也会出现

这么不起眼的数据结构，却有着红黑树和AVL树的性能(增删改查O(logn)，空间O(logn))、被Redis采用作为ZSet的底层数据结构，到底是如何实现的？关键是它的插入算法——利用正态分布。

当插入新的节点时，用抛硬币(随机函数)来决定是否增加层数，而这样的结果是节点所属层数正好符合 __正态分布，期望值是2__ 。

与红黑树和AVL树相比，逻辑简单、没有复杂的旋转平衡操作，可以很容易地用链表实现。

# 数据结构的实现
```	Golang
package skiplist

import (
	"math"
	"math/rand"
)

// SkipElement is the element for Skiplist
type SkipElement struct {
	value interface{}
	seq   int
	next  *SkipElement
	down  *SkipElement
}

// SkipList is a multilevel list. All operate time complexty is O(logn)
type SkipList struct {
	lists []*SkipElement
}

// NewList create a SkipList and return the pointer
func NewList() *SkipList {
	return &SkipList{
		lists: []*SkipElement{
			&SkipElement{
				seq: math.MinInt32,
			},
		},
	}
}

// Add or update value of a key
func (p *SkipList) Add(key int, val interface{}) {
	level := 0
	// get insert level by random
	for (rand.Int() & 1) == 1 {
		level++
	}

	for pre := p.lists[0]; len(p.lists) <= level; p.lists = append(p.lists, pre) {
		pre = &SkipElement{
			seq:  math.MinInt32,
			down: pre,
		}
	}

	var upNode *SkipElement
	for curLevel, curNode := level, p.lists[level]; curLevel >= 0 && curNode != nil; curLevel, curNode = curLevel-1, curNode.down {

		for curNode.next != nil && curNode.next.seq < key {
			curNode = curNode.next
		}

		// already exists, update value
		if curNode.next != nil && curNode.next.seq == key {
			for tempNode := curNode; tempNode != nil; tempNode = tempNode.down {
				tempNode.value = val
			}
			return
		}

		newNode := &SkipElement{
			seq:   key,
			value: val,
			next:  curNode.next,
		}
		if upNode != nil {
			upNode.down = newNode
		}
		curNode.next = newNode
		upNode = newNode
	}
}

// Find the element, if not found return nil
func (p *SkipList) Find(key int) (res interface{}) {
	for curLevel, curNode := len(p.lists)-1, p.lists[len(p.lists)-1]; curLevel >= 0 && curNode != nil; curLevel, curNode = curLevel-1, curNode.down {
		for curNode.next != nil && curNode.next.seq <= key {
			curNode = curNode.next
		}

		if curNode.seq == key {
			res = curNode.value
			return
		}
	}
	return
}

// Remove the element
func (p *SkipList) Remove(key int) {
	for curLevel, curNode := len(p.lists)-1, p.lists[len(p.lists)-1]; curLevel >= 0 && curNode != nil; curLevel, curNode = curLevel-1, curNode.down {
		for curNode.next != nil && curNode.next.seq < key {
			curNode = curNode.next
		}

		if curNode.next.seq == key {
			curNode.next = curNode.next.next
		}
	}
	return
}
```

# 待改进
* 增加迭代器，可以从某元素进行迭代
* 增加向前的指针，可以双向迭代
* 真正的value应该作为卫星数据存储在最底层，Find时返回最底层元素的迭代器
* Add函数应该返回一个bool型，表示是否是首次插入
* Remove函数也返回bool型，表示是否存在

# 应用示例
```	Golang
// 暂无
```

