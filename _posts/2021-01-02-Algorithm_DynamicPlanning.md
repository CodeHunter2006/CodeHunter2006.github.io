---
layout: post
title: "Algorithm Dynamic Planning"
date: 2021-01-02 09:00:00 +0800
tags: Algorithm Leetcode
---

记录 DP 的算法实现

### "78. Subsets" C++

```C++
vector<vector<int>> res;
for (int i = 0; i < nums.size(); ++i) {
    int size = res.size();
    for (int j = 0; j < size; ++j) {
        res.push_back(res[j]);
        res.back().push_back(nums[i]);
    }
}
return res;
```
