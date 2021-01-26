---
layout: post
title: "Algorithm Dynamic Planning"
date: 2021-01-02 09:00:00 +0800
tags: Algorithm Leetcode
---

记录 DP 的算法实现

# 算法说明

动态规划常常适用于有重叠子问题和最优子结构性质的问题，动态规划方法所耗时间往往远少于朴素解法。

- 优点：
  - 通常许多子问题非常相似，为此动态规划法试图仅仅解决每个子问题一次，具有天然剪枝的功能，从而减少计算量
  - 一旦某个给定子问题的解已经算出，则将其记忆化存储，以便下次需要同一个子问题解之时直接查表。这种做法在重复子问题的数目关于输入的规模呈指数增长时特别有用。

## 基本算法

- 思路：
  - 最终结果是由一系列状态转移逐步得出的，状态转移是有顺序的，后面的状态可能性可由前面的状态推出
  - 中间状态需要某种容器保存(dp)，容器容量通常要大一些用来保存初始状态
  - 每一次状态转移都可以获得一次局部最优解
  - 局部最优解转移到下一步未必是全局最优解，但局部最优解一定是上一步局部最优解转移而来
  - 动态规划状态转移时的算法核心是**状态转移方程**，用来描述`dp[i]`是如何由前面的 dp 演化而来的

## 零一背包问题

- 题目：
  有一个背包体积是 V，有 n 个物品体积为`v[i]`价值为`p[i]`，问如何把最大价值的物品放入背包
- 思路：
  - 在**零一背包**问题中选取某物品后就不能再选，这就是**后效性**，是指做了某种选择后会影响后面的状态决策。
  - 动态规划当中因为无法判断当前枚举的状态的来源，所以不允许出现后效性，如果解决不了则不能使用动态规划。
  - 由于**后效性**会导致状态迁移发生逻辑冲突，比如"拿过的物品下次还会再被选择"，所以我们要想办法消除后效性。
    解决方法需要保证两个顺序：
    1. **先遍历决策再遍历状态**保证每个决策只在每个状态上最多应用一次，比如"一个物品只拿一次"
    2. 如果同层状态会影响"后面的"状态，则利用**倒序**消除这种影响
  - 无**后效性**的例子比如"用最少数量的硬币凑够目标值"，用过的硬币不影响以后再用。不过同样要保证演进方向，这是 dp 算法的基础。

# 示例

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

### "139. Word Break"

```Go
func wordBreak(s string, wordDict []string) bool {
    m := make(map[string]bool, len(wordDict))
    for _, s := range wordDict {
        m[s] = true
    }
    dp := make([]bool, len(s)+1)
    dp[0] = true
    for i := 1; i <= len(s); i++ {
        for _, w := range wordDict {
            if len(w) <= i && string(s[i-len(w):i])==w && dp[i-len(w)] {
                dp[i] = true
                break
            }
        }
    }
    return dp[len(s)]
}
```

### "416. Partition Equal Subset Sum" Golang

```Go
func canPartition(nums []int) bool {
    n := len(nums)
    if n < 2 {
        return false
    }

    sum, max := 0, 0
    for _, v := range nums {
        sum += v
        if v > max {
            max = v
        }
    }
    if sum & 1 == 1 {
        return false
    }

    target := sum / 2
    if max > target {
        return false
    }

    // dp[i][j] 表示考虑第i个元素时，是否可以达到j这个总数
    // dp[i][j] = 不取i | 取i = dp[i-1][j] | dp[i-1][j-v]
    // 将dp精简为一维数组dp[j]，0 <= j <= target，其中 0 表示"什么都不取"所以一定为 true
    dp := make([]bool, target+1)
    dp[0] = true
    for i := 0; i < len(nums); i++ {
        for j := target; j >= 0; j-- {
            if j - nums[i] >= 0 {
                dp[j] = dp[j] || dp[j-nums[i]]
            }
        }
    }

    return dp[target]
}
```
