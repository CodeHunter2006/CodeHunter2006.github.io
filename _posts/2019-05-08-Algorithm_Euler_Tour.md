---
layout: post
title:  "算法 欧拉路径(Euler Tour)"
date:   2019-05-08 10:00:00 +0800
tags: Algorithm
---
### 算法特性
欧拉图（Euler Graph）是指能够满足欧拉路径/环游(Euler Tour)的有限有向图。<br/>
欧拉路径是指有限有向图中，经过所有节点，每个节点只被访问一次的路径。<br/>

![七桥问题](/assets/Euler_Tour.jpg)<br/>
欧拉提出：哥尼斯堡七桥问题

满足欧拉图的条件：
* 1 图是连通的，图上任意两点总有路径相连。
* 下面满足两者之一就可以：
	* 2.1 有且只有一个点的入度比出度少1（作为欧拉路径的起点），有且只有一个点的入度比出度多1（作为终点），且其余点入度=出度。
	* 2.2 所有点入度=出度。
	
<br/>
### 算法流程(Hierholzer算法)
* 1.在满足条件的欧拉图的基础上，从出发点开始出发。
* 2.向任意一个点出发，把当前路径删除，将起点插入结果数组头部，把目标点定为新的起点。
* 3.不断重复2操作，直到结束，那么输出数组就是欧拉路径了。

<br/>
### 算法的应用示例
``` CPP
Challenge:
Given a list of airline tickets represented by pairs of departure
and arrival airports [from, to], reconstruct the itinerary in order.
All of the tickets belong to a man who departs from JFK.
Thus, the itinerary must begin with JFK.

Note:
If there are multiple valid itineraries, you should return the itinerary
that has the smallest lexical order when read as a single string.
For example, the itinerary ["JFK", "LGA"] has a smaller lexical order than ["JFK", "LGB"].
All airports are represented by three capital letters (IATA code).
You may assume all tickets form at least one valid itinerary.


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