---
layout: post
title: "Algorithm Graph"
date: 2021-05-02 09:00:00 +0800
tags: Algorithm Leetcode
---

记录 Graph 的算法实现

### "277. Find the Celebrity"

```C++
for (int i = 0, j = 0; i < n; i++) {
    for (j = 0; j < n; j++) {
        if (i != j && knows(i, j)) break; //if i knows j, i is not celebrity
        if (i != j && !knows(j, i)) break; //if j don't know i, not celebrity
    }
    if (j == n) return i; //i does not know any j , but all j know i
}
return -1;
```

### "310. Minimum Height Trees"

```C++
vector<int> findMinHeightTrees(int n, vector<vector<int>>& edges) {
    if (n == 1) return {0};
    vector<unordered_set<int>> g(n);
    for (auto &e : edges) {
        g[e[0]].insert(e[1]);
        g[e[1]].insert(e[0]);
    }
    queue<int> q;
    for (int i = 0; i < n; ++i) {
        if (g[i].size() == 1)
            q.push(i);
    }
    while (n > 2) {
        int size = q.size();
        n -= size;
        for (int i = 0; i < size; ++i) {
            int e = q.front(); q.pop();
            for (auto a : g[e]) {
                g[a].erase(e);
                if (g[a].size() == 1)
                    q.push(a);
            }
        }
    }
    vector<int> res;
    while (q.size()) {
        res.push_back(q.front());
        q.pop();
    }
    return res;
}
```

### "332. Reconstruct Itinerary"

```C++
unordered_map<string,multiset<string>> m_;	// tickets map
vector<string> res_;
void dfs(string start) {
    while (m_[start].size()) {
        string next = *m_[start].begin();
        // 注意，这里是multiset，所以erase(iterator)只会删除一个元素。
        m_[start].erase(m_[start].begin());
        dfs(next);
    }
    // 按照Hierholzer算法，应该插入到最左边，所以res最后要reverse一下。
    res_.push_back(start);
}
vector<string> findItinerary(vector<vector<string>>& tickets) {
    for (auto i : tickets) {
        m_[i[0]].insert(i[1]);
    }
    dfs("JFK");
    // 按照Hierholzer算法，应该插入到最左边，所以res最后要reverse一下。
    reverse(res_.begin(), res_.end());
    return res_;
}
```

### "743. Network Delay Time"

```Go
// Dijkstra
func networkDelayTime(times [][]int, n int, k int) int {
    dist := make([]int, n)
    for i := range dist {
        dist[i] = math.MaxInt32
    }
    edges := make(map[int][][2]int)
    for _, t := range times {
        edges[t[0]] = append(edges[t[0]], [2]int{t[1], t[2]})
    }

    h := &myHeap{[2]int{k, 0}}
    for h.Len() > 0 {
        edge := heap.Pop(h).([2]int)
        if dist[edge[0]-1] == math.MaxInt32 {
            for _, w := range edges[edge[0]] {
                heap.Push(h, [2]int{w[0], edge[1]+w[1]})
            }
        }
        if dist[edge[0]-1] > edge[1] {
            dist[edge[0]-1] = edge[1]
        }
    }

    maxVal := 0
    for _, d := range dist {
        if d == math.MaxInt32 {
            return -1
        } else if d > maxVal {
            maxVal = d
        }
    }

    return maxVal
}

type myHeap [][2]int // [vertex,dist]
func (p myHeap) Len() int {
    return len(p)
}
func (p myHeap) Less(a, b int) bool {
    return p[a][1] < p[b][1]
}
func (p *myHeap) Swap(a, b int) {
    (*p)[a], (*p)[b] = (*p)[b], (*p)[a]
}
func (p *myHeap) Push(x interface{}) {
    *p = append(*p, x.([2]int))
}
func (p *myHeap) Pop() interface{} {
    n := len(*p)-1
    tmp := (*p)[n]
    (*p) = (*p)[:n]
    return tmp
}
```

```Go
// Bellmen-Ford
func networkDelayTime(times [][]int, n int, k int) int {
    dp := make([]int, n)
    for i := range dp {
        dp[i] = math.MaxInt32
    }
    dp[k-1] = 0
    for i := 1; i < n; i++ {
        for _, t := range times {
               dp[t[1]-1] = min(dp[t[1]-1], dp[t[0]-1]+t[2])
        }
    }

    maxVal := 0
    for _, v := range dp {
        if v == math.MaxInt32 {
            return -1   // 有不可达结点
        } else if v < 0 {
            return -1   // 存在负权边环
        } else if v > maxVal {
            maxVal = v
        }
    }

    return maxVal
}
```

```Go
// Floyd-Warshall
func networkDelayTime(times [][]int, n int, k int) int {
    matrix := make([][]int, n)
    for i := 0; i < n; i++ {
        matrix[i] = make([]int, n)
        for j := range matrix[i] {
            matrix[i][j] = math.MaxInt32
        }
        matrix[i][i] = 0
    }
    for _, t := range times {
        matrix[t[0]-1][t[1]-1] = t[2]
    }

    for k := 0; k < n; k++ {
        for i := 0; i < n; i++ {
            for j := 0; j < n; j++ {
                if matrix[i][j] > matrix[i][k] + matrix[k][j] {
                    matrix[i][j] = matrix[i][k] + matrix[k][j]
                }
            }
        }
    }

    maxVal := 0
    for i := 0; i < n; i++ {
        if matrix[k-1][i] == math.MaxInt32 {
            return -1
        } else if matrix[k-1][i] > maxVal {
            maxVal = matrix[k-1][i]
        }
    }

    return maxVal
}
```

### "787. Cheapest Flights Within K Stops"

```Go
// Bellmen-Ford
```

### "1192. Critical Connections in a Network"

```Go
func criticalConnections(n int, connections [][]int) (ret [][]int) {
    seq := 0
    dfn, low, graph := make([]int, n), make([]int, n), make([][]int, n)
    for i := range dfn {
        dfn[i] = -1
    }
    for _, c := range connections {
        graph[c[0]] = append(graph[c[0]], c[1])
        graph[c[1]] = append(graph[c[1]], c[0])
    }

    var dfs func(int, int)
    dfs = func(cur, from int) {
        seq++
        dfn[cur], low[cur] = seq, seq
        fmt.Println("in", cur, seq, seq)
        for _, next := range graph[cur] {
            if next == from {
                continue    // 避免死循环
            }
            if dfn[next] == -1 {
                dfs(next, cur)
                if low[next] > dfn[cur] {
                    ret = append(ret, []int{cur, next})
                }
            }
            fmt.Println("low", cur, low[cur], low[next])
            low[cur] = min(low[cur], low[next])
        }
    }

    // 由于题目是联通图，所以只需要从任意一点出发
    dfs(0, 0)
    return ret
}
```
