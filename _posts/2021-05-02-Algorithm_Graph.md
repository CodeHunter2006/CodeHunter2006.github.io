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
