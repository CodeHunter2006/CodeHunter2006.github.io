---
layout: post
title: "Algorithm OrderedMap"
date: 2021-01-02 09:00:00 +0800
tags: Algorithm Leetcode
---

记录 Ordered Map 的算法实现

- Ordered Map 容器有下面特性：

  - 元素在容器中有序排列
  - 可以在 O(logn)内进行一次 CRUD 操作
  - 找到元素后可以通过迭代器快速找到前/后元素

- 可以通过多种方式实现 Ordered Map:
  - BST(Binary Search Tree) (实现简单、由于没有平衡能力性能差)
  - AVL-Tree(平衡二叉树) (实现相对简单、性能可以)
  - Treap(堆树) (实现相对简单、性能可以)
  - Red Black Tree (实现复杂、性能较好)
  - Skip List (实现相对简单、性能可以)

### "220. Contains Duplicate III"

```C++
// Red Black Tree
class Solution {
public:
    bool containsNearbyAlmostDuplicate(vector<int>& nums, int k, int t) {
        int n = nums.size();
        set<int> rec;
        for (int i = 0; i < n; i++) {
            auto iter = rec.lower_bound(max(nums[i], INT_MIN + t) - t);
            if (iter != rec.end() && *iter <= min(nums[i], INT_MAX - t) + t) {
                return true;
            }
            rec.insert(nums[i]);
            if (i >= k) {
                rec.erase(nums[i - k]);
            }
        }
        return false;
    }
};
```

```Go
// Treap
import "math/rand"

type node struct {
    ch       [2]*node
    priority int
    val      int
}

func (o *node) cmp(b int) int {
    switch {
    case b < o.val:
        return 0
    case b > o.val:
        return 1
    default:
        return -1
    }
}

func (o *node) rotate(d int) *node {
    x := o.ch[d^1]
    o.ch[d^1] = x.ch[d]
    x.ch[d] = o
    return x
}

type treap struct {
    root *node
}

func (t *treap) _put(o *node, val int) *node {
    if o == nil {
        return &node{priority: rand.Int(), val: val}
    }
    d := o.cmp(val)
    o.ch[d] = t._put(o.ch[d], val)
    if o.ch[d].priority > o.priority {
        o = o.rotate(d ^ 1)
    }
    return o
}

func (t *treap) put(val int) {
    t.root = t._put(t.root, val)
}

func (t *treap) _delete(o *node, val int) *node {
    if d := o.cmp(val); d >= 0 {
        o.ch[d] = t._delete(o.ch[d], val)
        return o
    }
    if o.ch[1] == nil {
        return o.ch[0]
    }
    if o.ch[0] == nil {
        return o.ch[1]
    }
    d := 0
    if o.ch[0].priority > o.ch[1].priority {
        d = 1
    }
    o = o.rotate(d)
    o.ch[d] = t._delete(o.ch[d], val)
    return o
}

func (t *treap) delete(val int) {
    t.root = t._delete(t.root, val)
}

func (t *treap) lowerBound(val int) (lb *node) {
    for o := t.root; o != nil; {
        switch c := o.cmp(val); {
        case c == 0:
            lb = o
            o = o.ch[0]
        case c > 0:
            o = o.ch[1]
        default:
            return o
        }
    }
    return
}

func containsNearbyAlmostDuplicate(nums []int, k, t int) bool {
    set := &treap{}
    for i, v := range nums {
        if lb := set.lowerBound(v - t); lb != nil && lb.val <= v+t {
            return true
        }
        set.put(v)
        if i >= k {
            set.delete(nums[i-k])
        }
    }
    return false
}
```

### "363. Max Sum of Rectangle No Larger Than K"

```Go
func maxSumSubmatrix(matrix [][]int, k int) (ret int) {
    m, n := len(matrix), len(matrix[0])
    for i := 0; i < m; i++ {
        sum := make([]int, n)
        for j := i; j < m; j++ {
            for k := 0; k < n; k++ {
                sum[k] += matrix[j][k]
            }
            t := &treap{root:&node{}}
            s := 0
            for _, v := range sum {
                s += v
                if o := t.lowerBound(s - k); o != nil {
                    ret = max(ret, s - o.val)
                }
                t.put(s)
            }
        }
    }
    return ret
}
```
