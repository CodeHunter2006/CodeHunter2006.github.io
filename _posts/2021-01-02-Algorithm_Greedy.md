---
layout: post
title: "Algorithm Greedy"
date: 2021-01-02 09:00:00 +0800
tags: Algorithm Leetcode
---

记录 Greedy 的算法实现

### "406. Queue Reconstruction by Height" C++

```C++
sort(people.begin(), people.end(),[](pair<int,int> p1, pair<int,int> p2){
    return p1.first > p2.first || (p1.first == p2.first && p1.second < p2.second);
});
vector<pair<int,int>> sol;
for (auto person : people)
    sol.insert(sol.begin() + person.second, person);
return sol;
```

```Golang
type myPeople [][]int

func (p myPeople) Len() int {
    return len(p)
}

func (p myPeople) Less(a, b int) bool {
    return p[a][0] > p[b][0] || (p[a][0] == p[b][0] && p[a][1] < p[b][1])
}

func (p myPeople) Swap(a, b int) {
    p[a], p[b] = p[b], p[a]
}

func reconstructQueue(people [][]int) (ret [][]int) {
    sort.Sort(myPeople(people))

    for i := range people {
        ret = append(ret[:people[i][1]], append([][]int{people[i]}, ret[people[i][1]:]...)...)
    }

    return ret
}
```
