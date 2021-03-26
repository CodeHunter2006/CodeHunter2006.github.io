---
layout: post
title: "算法学习——Dijkstra"
date: 2019-02-18 10:00:00 +0800
tags: Algorithm
---

![Edsger Wybe Dijkstra](/assets/images/2019-02-18-Algorithm_Dijkstra_1.jpg)

# 吐槽

Dijkstra 翻译为"迪杰特斯拉"，但是音标是/ˈdaɪkstrə/，应该是"戴克特斯拉"才对呀。不过/ai/ /i/只是嘴巴大小不同，可能是传过来时被别的语言音译了。

Go 中没有像 C++ STL 那么多的常用容器，不过 list 可以作为 stack、queue 来用，Go 提供的 heap 需要自己实现一些接口从而实现 priority_queue 的功能。希望 Go 能支持泛型，提供更多容器。

# 算法特性

Dijkstra<br/>
用于求出图中某节点到其他每个节点的最短路径，主要利用 PriorityQueue 数据结构。时间复杂度 O((E+N)lgN)，空间复杂度 O(E)，E 是边的数量、N 是节点数量。

# 算法流程

- 1 创建一个数组，存储起始节点到所有目标节点的距离，默认存储无穷大。
- 2 创建一个优先级队列(距离越小越靠前)，存储到达某个节点时的(出发点到这点的距离，当前节点 ID)，默认放入出发节点(距离 0)。
- 3 弹出一个距离最小的节点，判断是否需要更新距离数组。将所有子节点插入优先级队列同时插入距离累加值。然后在图中删除当前节点为出发点的边。
- 4 循环 3，直到所有队列元素处理完毕。可以提前判断是否所有元素已经处理过，因为距离数组第一次被赋值总是最短的距离。
- 5 遍历距离数组，如果还有无穷大内容，说明有节点无法达到；如果都能达到，则可以记录最大值返回，代表并行出发时的最短距离(或时间)。

# 算法的应用示例

```
challenge:
There are N network nodes, labelled 1 to N.

Given times, a list of travel times as directed edges times[i] = (u, v, w),
where u is the source node, v is the target node,
and w is the time it takes for a signal to travel from source to target.

Now, we send a signal from a certain node K.
How long will it take for all nodes to receive the signal? If it is impossible, return -1.

type TarDist struct {
    target, distance int    // target node ID, distance from node K
}

type DistHeap [](*TarDist)

func (h DistHeap) Len() int {
    return len(h)
}

func (h DistHeap) Less(a, b int) bool {
    return h[a].distance < h[b].distance
}

func (h DistHeap) Swap(a, b int) {
    h[a], h[b] = h[b], h[a]
}

func (h *DistHeap) Push(x interface{}) {
    *h = append(*h, x.(*TarDist))
}

func (h *DistHeap) Pop() (x interface{}) {
    x = (*h)[len(*h)-1]
    (*h) = (*h)[:len(*h)-1]
    return
}

func networkDelayTime(times [][]int, N int, K int) int {
    // store the edges to map
    edges := make(map[int]map[int]int)
    for _, t := range times {
        if _, exists := edges[t[0]]; !exists {
            edges[t[0]] = make(map[int]int)
        }
        edges[t[0]][t[1]] = t[2]
    }

    // prepare distance record from K to every node
    dist := make([]int, N)
    for i := range dist {
        dist[i] = math.MaxInt32
    }

    // push K for priority queue initiate
    foundCount := 0
    pq := &DistHeap{}
    heap.Push(pq, &TarDist{
        target : K,
        distance : 0,
    })

    for pq.Len() > 0 {
        cur := heap.Pop(pq).(*TarDist)

        // record count of nodes that can be reached
        if dist[cur.target-1] == math.MaxInt32 {
            foundCount++
        }

        if dist[cur.target-1] <= cur.distance {
            continue
        }

        dist[cur.target-1] = cur.distance
        if foundCount == N {
            // avoid useless search
            break
        }
        for key, value := range edges[cur.target] {
            heap.Push(pq, &TarDist{
                target : key,
                distance : cur.distance + value,
            })
        }
        delete(edges, cur.target)
    }

    // some node can't be reached
    if foundCount < N {
        return -1
    }

    // count the max distance from K
    maxDist := 0
    for _, d := range dist {
        if maxDist < d {
            maxDist = d
        }
    }

    return maxDist
}
```

# LeetCode 题目

### "818. Race Car"

```Go
func racecar(target int) int {
    K := 32 - bits.LeadingZeros32(uint32(target))
    MAX_RANGE := (1<<K)-1       // 边界距离
    dist := make(map[int]int)   // 每个位置的最小步数
    h := new(myheap)

    dist[0] = 0
    heap.Push(h, [2]int{0, 0})
    for h.Len() > 0 {
        tmp := heap.Pop(h).([2]int)
        steps1, tar1 := tmp[0], tmp[1]
        if val, exists := dist[tar1]; exists && val < steps1 {
            continue
        }
        for k := 0; k <= K; k++ {
            tar2 := (1<<k)-1-tar1       // 在这里调头了
            steps2 := steps1 + k + 1
            if tar2 == target {
                steps2--                // 去掉最后的R
            }
            if val, exists := dist[tar2]; abs(tar2)<=MAX_RANGE && (exists && val>steps2 || !exists) {
                heap.Push(h, [2]int{steps2, tar2})
                dist[tar2] = steps2
            }
        }
    }

    return dist[target]
}

func abs(a int) int {
    if a < 0 {
        return -a
    }
    return a
}

type myheap [][2]int
func (p myheap) Len() int {
    return len(p)
}
func (p myheap) Less(a, b int) bool {
    return p[a][0] < p[b][0]
}
func (p myheap) Swap(a, b int) {
    p[a], p[b] = p[b], p[a]
}
func (p *myheap) Push(val interface{}) {
    *p = append(*p, val.([2]int))
}
func (p *myheap) Pop() interface{} {
    len := p.Len()
    tmp := (*p)[len-1]
    *p = (*p)[:len-1]
    return tmp
}
```
