---
layout: post
title: "Algorithm Minimax"
date: 2021-05-02 09:00:00 +0800
tags: Algorithm Leetcode
---

记录 Minimax 的算法实现

# 模板

- 以"292. Nim Game"为例的 Minimax 模板

```java
// dfs + memo
private boolean canFirstWin(int n) {
    Boolean[] memo = new Boolean[n+1];  // 初始化 memo
    return dfs(n, memo);
}

private boolean dfs(int n, Boolean[] memo) {
    if (n < 0) return false;                // 超出边界
    if (memo[n] != null) return memo[n];    // 已有结果
    boolean res = false;                    // 初始化为失败
    for (int i = 1; i < 4; i++)             // 核心递推公式
        if !dfs(n-i, memo) {                // 找到任意对方 min 的情况
            res = true;
            break;
        }
    return memo[n] = res;                   // 保存到 memo 并返回
}
```

- 每次 dfs 都是向更低数量的子集取得答案，因此可以用 DP 实现，用 DP 时需要考虑演化矩阵边界
- 只要从子集中找到任何自己 max 或对方 min 的情况，就算自己 win
- memo 可以是一维或二维
- 遍历子集情况时，是将当前可做的决策遍历一遍，同时检查决策后对手子集的情况
- 遍历子集前要初始化当前结果，注意要初始化为合理的值(例如`math.MinInt32`)，避免影响后面的"取最优解"运算
- 如果是计算分值，计算过程中只要记录两者之差可以简化操作，如果最后需要值，只需要根据两者之差和所有元素和计算二元一次方程即可

# 题目

### "486. Predict the Winner"

```Go
func PredictTheWinner(nums []int) bool {
    memo := make([][]int, len(nums))
    for i := range memo {
        memo[i] = make([]int, len(nums))
    }
    return dfs(nums, 0, len(nums)-1, memo) >= 0
}

func dfs(nums []int, l, r int, memo [][]int) int {
    if l == r {
        return nums[l]
    } else if memo[l][r] != 0 {
        return memo[l][r]
    }
    memo[l][r] = max(nums[l]-dfs(nums, l+1, r, memo), nums[r]-dfs(nums, l, r-1, memo))
    return memo[l][r]
}
```

### "913. Cat and Mouse"

```C++
int calc(int curM, int curC, int isM, vector<vector<vector<int>>>& dp, vector<vector<int>>& graph) {
    if(curC==0 || curM==0)
        return 1;
    if(curC==curM)
        return 2;
    if(dp[curM][curC][isM]!=-1) {
        return dp[curM][curC][isM];
    }
    dp[curM][curC][isM]=0;
    bool canDraw=false;
    if(isM) {
        for(int i=0;i<graph[curM].size();i++) {     //If any neigbor is 0 return 1
            if(graph[curM][i]==0) {
                dp[curM][curC][isM]=1;
                return 1;
            }
        }
        for(int i=0;i<graph[curM].size();i++) {
            int temp=calc(graph[curM][i], curC, 0, dp, graph);
            if(temp==1) {
                dp[curM][curC][isM]=1;
                return 1;
            }
            else if(temp==0) {
                canDraw=true;
            }
        }
        if(!canDraw)
            dp[curM][curC][isM]=2;
    } else {
        for(int i=0;i<graph[curC].size();i++) {     //If any neighbor is current pos of mouse return 2
            if(graph[curC][i]==curM) {
                dp[curM][curC][isM]=2;
                return 2;
            }
        }
        for(int i=0;i<graph[curC].size();i++) {
            int temp=calc(curM, graph[curC][i], 1, dp, graph);
            if(temp==2) {
                dp[curM][curC][isM]=2;
                return 2;
            }
            else if(temp==0) {
                canDraw=true;
            }
        }
        if(!canDraw)
            dp[curM][curC][isM]=1;
    }
    return dp[curM][curC][isM];
}

int catMouseGame(vector<vector<int>>& graph) {
    int n=graph.size();
    vector<vector<vector<int>>> dp(n, vector<vector<int>>(n, vector<int>(2, -1)));
    return calc(1, 2, 1, dp, graph);
}
```

### "1140. Stone Game II"

```Go
func stoneGameII(piles []int) int {
    n := len(piles)
    memo := make([][]int, n)
    for i := range memo {
        memo[i] = make([]int, (n+1)>>1)
    }
    postSum := make([]int, n)
    for sum, i := 0, len(piles)-1; i >= 0; i-- {
        sum += piles[i]
        postSum[i] = sum
    }
    diff := dfs(n, 0, 1, postSum, memo)
    return (diff + postSum[0])/2
}

func dfs(n, start, m int, postSum []int, memo [][]int) (ret int) {
    if start + 2*m >= n {
        return postSum[start]
    } else if memo[start][m] > 0 {
        return memo[start][m]
    }
    val := math.MinInt32
    for i := 1; i <= 2*m; i++ {
        val = max(val, postSum[start]-postSum[start+i] - dfs(n, start+i, max(i, m), postSum, memo))
    }
    memo[start][m] = val
    return val
}
```
