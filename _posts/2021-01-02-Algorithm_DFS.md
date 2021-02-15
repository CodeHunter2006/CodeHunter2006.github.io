---
layout: post
title: "Algorithm DFS"
date: 2021-01-02 09:00:00 +0800
tags: Algorithm Leetcode
---

记录 SFS 的算法实现

### "377. Combination Sum IV" C++ DFS

```C++
int dfs(vector<int>& dp, int target, vector<int>& nums) {
    if (target == 0)
        return 1;
    if (dp[target] != -1)
        return dp[target];
    int sum = 0;
    for (auto num : nums) {
        if (target >= num) {
            sum += dfs(dp, target - num, nums);
        }
    }
    return dp[target] = sum;
}
int combinationSum4(vector<int>& nums, int target) {
    vector<int> dp(target + 1, -1);
    return dfs(dp, target, nums);
}
```
