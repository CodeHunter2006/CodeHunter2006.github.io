---
layout: post
title: "Algorithm DFS"
date: 2021-01-02 09:00:00 +0800
tags: Algorithm Leetcode
---

记录 SFS 的算法实现

### "91. Decode Ways"

```Go
func numDecodings(s string) (ret int) {
    m := map[string]bool{}
    for i := 1; i <= 26; i++ {
        m[strconv.Itoa(i)] = true
    }
    mem := map[int]int{}
    var recur func(string, int) int
    recur = func(s string, idx int) (ret int) {
        if idx == len(s) {
            return 1
        }
        if val, ok := mem[idx]; ok{
            return val
        }
        if m[string(s[idx])] {
            ret += recur(s, idx+1)
        }
        if idx+1 < len(s) && m[string(s[idx:idx+2])] {
            ret += recur(s, idx+2)
        }
        mem[idx] = ret
        return ret
    }
    return recur(s, 0)
}
```

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
