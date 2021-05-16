---
layout: post
title: "Algorithm Greedy"
date: 2021-01-02 09:00:00 +0800
tags: Algorithm Leetcode
---

记录 Greedy 的算法实现

### "321. Create Maximum Number"

```C++
vector<int> maxNumber(vector<int>& nums1, vector<int>& nums2, int k) {
    const int n1 = nums1.size(), n2 = nums2.size();
    vector<int> best;
    for (int k1 = max(k - n2, 0); k1 <= min(k, n1); ++k1)
        best = max(best, maxNumber(maxNumber(nums1, k1),
                                   maxNumber(nums2, k-k1)));
    return best;
}
vector<int> maxNumber(vector<int> nums, int k) {
    int drop = nums.size() - k;
    vector<int> out;
    for (int num : nums) {
        while (drop && out.size() && out.back() < num) {
            out.pop_back();
            drop--;
        }
        out.push_back(num);
    }
    out.resize(k);
    return out;
}
vector<int> maxNumber(vector<int> nums1, vector<int> nums2) {
    vector<int> out;
    auto i1 = nums1.begin(), end1 = nums1.end();
    auto i2 = nums2.begin(), end2 = nums2.end();
    while (i1 != end1 || i2 != end2)
        out.push_back(lexicographical_compare(i1, end1, i2, end2) ? *i2++ : *i1++);
    return out;
}
```

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

### "435. Non-overlapping Intervals"

```Go
func eraseOverlapIntervals(intervals [][]int) (count int) {
    sort.Slice(intervals, func(a, b int) bool {
        return intervals[a][1] < intervals[b][1]
    })
    for i, end := 0, math.MinInt32; i < len(intervals); i++ {
        if intervals[i][0] >= end {
            end = intervals[i][1]
        } else {
            count++
        }
    }
    return count
}
```
