---
layout: post
title:  "算法学习，拓扑排序(Topological Order)"
date:   2018-08-01 10:00:00 +0800
tags: Algorithm
---
### 应用场景
如果数据关系可以形成一个有向无环图，那么可以用拓扑排序形成一个唯一顺序序列，BFS是比较常用的方法。

例如：有一系列任务，有些任务有前后依赖关系，现在想知道用怎样的顺序可以把任务做完。

### 算法流程
* 1 选择一个入度为0的定点，输出它，并在图中删除。
* 2 删除这个定点对应的边。
* 3 不断循环1、2，直到结束。
* 4 如果输出的顶点数小于总顶点数，则说明有回路。否则，输出的序列就是拓扑排序结果。

### 算法示例
```
/*
  输入： 参数1 表示有n个节点。
       参数2 边的数组，每条边连接两个节点，每个节点以id号表示，0 <= id < n
            保证这些边可以组成一个Tree
		 
  输出： 哪些顶点如果作为根，生成的Tree可以最矮，输出到数组
	
  思路： 目标是找到与所有节点都连接总距离最近的节点，可以用拓扑排序，
       找到最中间的那些点正好符合要求。
       虽然题目是一个无向图，而拓扑排序是解决有向图问题的，
       但是可以看做是双向的有向图来处理。
*/
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