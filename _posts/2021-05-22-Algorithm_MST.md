---
layout: post
title: "Algorithm MST"
date: 2021-05-22 09:00:00 +0800
tags: Algorithm Leetcode
---

记录 MST 的算法实现

# 算法

MST(Minimum-cost Spanning Tree 最小生成树)是在图中连接各点生成一个树(无环、全连接)，使得树的边的长度总和最小。
MST 应用很广泛，比如道路、网络规划，以最小成本修建整个网络。

## 基本概念

- **V**Vertex，顶点数
- **E**Edge，边数

如果能够生成最小生成树，那必定是**连通图**，即所有顶点都可以到达

- **dense graph 稠密图**边数较多的图
  - 适合 Prim Naive 算法，O(V^2)
- **sparse graph 稀疏图**边数较少的图
  - 适合 Prim PQ / Kruskal UF 算法，O(ElogV)

## Prim

- 基本步骤：

  1. 从指定的顶点开始寻找最小权值的邻接点
  2. 不断的将下一个最小权值的边连接的顶点加入集合
  3. 直到所有顶点都加入到集合中

- Prim Naive implementation **O(V^2)**

  1. 初始化 visited 数组为 false，表示某点是否进入集合；初始化数组 dist 为 MaxInt，表示 MST 到达某点的距离，默认为不可达 MaxInt
  2. 遍历处理，直到全部进入集合，初始加入 0 点
  3. 此点进入集合的同时，对所有通过该顶点的点做 edge relaxiation，即如果有更近的距离则更新距离，同时记录下一个最近距离点
  4. 执行 2 3 直到结束

- **最常用方法**Prim PQ implementation **O(ElogV)**
  - 遍历中用 PriorityQueue 找到下一个最小权值的边的顶点

## Kruskal

- Kruskal UF implementation **O(ElogV ~ ElogE)**，`V <= E <= V^2`

- 步骤：

  1. 先实现一个 UnionFind，size 和 V 数量一致
  2. 将所有边放入一个数组，同时附带这条边两边的 V 下标
  3. 对边数组排序
  4. 由小到大遍历每条边，如果边两端的 V 的 root 不同，则计入 cost、同时将两个 V Union
  5. 直到处理完所有边，即可找到目标

- 优化：
  如果利用 rank，可以提高 Union 效率；并且通过 rank == size 可以提前退出避免不必要的遍历

# 题目

"1135. Connecting Cities With Minimum Cost"
"1168. Optimize Water Distribution in a Village"

### "1584. Min Cost to Connect All Points"

```Go
// Prim PQ 400ms
func minCostConnectPoints(points [][]int) (cost int) {
    dist := func(a, b int) int {
        return abs(points[a][0]-points[b][0])+abs(points[a][1]-points[b][1])
    }

    h := new(myHeap)
    heap.Push(h, &[2]int{0,0})
    m := make(map[int]bool)

    for h.Len() > 0 && len(m) < len(points) {
        cur := heap.Pop(h).(*[2]int)
        if m[cur[1]] {
            continue
        }
        cost += cur[0]
        m[cur[1]] = true
        for i := range points {
            if m[i] {
                continue
            }

            heap.Push(h, &[2]int{dist(cur[1], i), i})
        }
    }
    return cost
}

type myHeap []*[2]int
func (p *myHeap) Len() int { return len(*p) }
func (p *myHeap) Less(a, b int) bool { return (*p)[a][0] < (*p)[b][0] }
func (p *myHeap) Swap(a, b int) { (*p)[a], (*p)[b] = (*p)[b], (*p)[a] }
func (p *myHeap) Push(x interface{}) { (*p) = append(*p, x.(*[2]int)) }
func (p *myHeap) Pop() interface{} {
    n := len(*p)
    tmp := (*p)[n-1]
    (*p) = (*p)[:n-1]
    return tmp
}

func abs(x int) int {
    if x < 0 {
        return -x
    }
    return x
}
```

```Go
// Prim Naive
func minCostConnectPoints(points [][]int) (cost int) {
    n := len(points)
    s := make(map[int]struct{}, n)
    dist := make([]int, n)
    for i := 0; i < n; i++ {
        s[i] = struct{}{}
        dist[i] = math.MaxInt32
    }

    next := 0
    dist[next] = 0
    for len(s) > 0 {
        cur := next
        cost += dist[cur]
        delete(s, cur)
        var minDist int = math.MaxInt32
        for j := range s {
            dist[j] = min(dist[j], abs(points[cur][0]-points[j][0])+abs(points[cur][1]-points[j][1]))
            if minDist > dist[j] {
                minDist = dist[j]
                next = j
            }
        }
    }
    return cost
}
```

```Go
// Kruskal 548ms
func minCostConnectPoints(points [][]int) (cost int) {
    uf := NewUF(len(points))
    edges := make([]*[3]int, 0, len(points)*len(points))

    for i := 0; i < len(points); i++ {
        for j := 1+i; j < len(points); j++ {
            edges = append(edges, &[3]int{abs(points[i][0]-points[j][0])+abs(points[i][1]-points[j][1]),i,j})
        }
    }
    sort.Slice(edges, func(a, b int) bool {return edges[a][0] < edges[b][0]})

    for _, e := range edges {
        if uf.Find(e[1]) == uf.Find(e[2]) {
            continue
        }
        cost += e[0]
        uf.Union(e[1], e[2])
    }
    return cost
}

type UnionFind []int
func NewUF(size int) *UnionFind {
    ret := make([]int, size)
    for i := 0; i < size; i++ {
        ret[i] = i
    }
    return (*UnionFind)(&ret)
}
func (p *UnionFind) Find(x int) int {
    if (*p)[x] != x {
        (*p)[x] = p.Find((*p)[x])
    }
    return (*p)[x]
}
func (p *UnionFind) Union(a, b int) {
    (*p)[p.Find(b)] = (*p)[p.Find(a)]
}
```
