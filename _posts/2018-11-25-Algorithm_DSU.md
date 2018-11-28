---
layout: post
title:  "算法学习，极限数据结构之——并查集(DSU)"
date:   2018-11-25 10:00:00 +0800
tags: Algorithm
---
### 算法特性
并查集，DSU(Disjoint Set Union)，对应的算法也叫Union Find。<br/>
该数据结构用于将有关联的元素合并为多个集合，使用非常简单、实现逻辑也不复杂。<br/>
对数据进行紧密排列处理(将离散数据紧密排列后添加id)后空间复杂度O(n)，时间复杂度O(n)，性能非常好。

<br/>
### 数据结构的实现
``` C++
// C++版本
class DSU{
    private:
    vector<int> parent_; // 用于集合关联，默认每个元素自己是一个集合。
    vector<int> rank_;   // 将孩子较少的节点合并到节点较多的节点，减少查找次数。
    public:
    DSU(int N) {
        parent_.resize(N);
        for (int i = 0; i < N; i++) // 每个元素默认是自己的集合的根
            parent_[i] = i;
        rank_.resize(N, 1);    // rank默认数为1
    }
    int find(int x) { // 每次查找，都递归进行合并，避免层数过高。
        if (parent_[x] != x)
            parent_[x] = find(parent_[x]);
        return parent_[x];
    }        
    int find(int x, int &outSize) {    // 可以查找当前集合总数
        if(parent_[x] != x) parent_[x] = find(parent_[x]);
        outSize = rank_[parent_[x]];
        return parent_[x];
    }
    bool doUnion(int x, int y) {
        int xr = find(x), yr = find(y);
        if(xr == yr)
            return false;
        else if (rank_[xr] < rank_[yr]) {
            parent_[xr] = yr;
            rank_[yr] += rank_[xr];
        } else {
            parent_[yr] = xr;
            rank_[xr] += rank_[yr];
        }
        return true;
    }
};
```

``` Golang
// Go 版本
type DSU struct {
    parents []int
    ranks []int
}

// 创建DSU对象
func NewDSU(size int) (res *DSU) {
    if size <= 0 {
        return
    }
    
    res = &DSU{
        parents : make([]int, size),
        ranks : make([]int, size),
    }
    
    for i := 0; i < size; i++ {
        res.parents[i] = i
        res.ranks[i] = 1
    }
    return
}

func (p *DSU) Find(x int) (root int, err error) {
    if x < 0 || x >= len(p.parents) {
        err = errors.New("invalid index")
        return
    }
    
    if p.parents[x] != x {
        p.parents[x], _ = p.Find(p.parents[x])
    }
    root = p.parents[x]
    return
}

func (p *DSU) Union(x ,y int) (res bool, err error) {
    if x < 0 || x >= len(p.parents) || y < 0 || y >= len(p.parents) {
        err = errors.New("invalid index")
        return
    }
    
    xRoot, _ := p.Find(x)
    yRoot, _ := p.Find(y)
    
    if xRoot == yRoot {
        res = false
        return
    }
    
    if p.ranks[xRoot] >= p.ranks[yRoot] {
        p.parents[yRoot] = xRoot
        p.ranks[xRoot] += p.ranks[yRoot]
    } else {
        p.parents[xRoot] = yRoot
        p.ranks[yRoot] += p.ranks[xRoot]        
    }
    res = true
    return
}
```

<br/>
### 应用示例
```
// 暂时缺少
```

