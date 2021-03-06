---
layout: post
title: "花花酱 DP Note"
date: 2021-03-06 09:00:00 +0800
tags: Algorithm Leetcode
---

花花酱 DP 算法模板笔记

- DP 基础可以参考[Algorithm Dynamic Planning](/2021/01/02/Algorithm_DynamicPlanning/)

## DP 的定义

- DP(Dynamic Planning) is aprogramming method.

  1. **optimal substructure**
     当前最优解由多个子最优解推导得出
  2. **overlapping sub-problems**
     子问题会被重复计算，使用 DP 可以避免重复计算从而降低时间复杂度
  3. **No-after effect**
     无后效性，子问题的最优解一旦确定，在计算后面问题时就不需要再改变

- DP 通常可以由"recur+memory"推导而来，DP 的优势是通过定向推导避免重复计算实现了降维

## 分类表

| Case   | Input     | Subproblems | Depends on | Time    | Space         | Problem IDs                   |
| ------ | --------- | ----------- | ---------- | ------- | ------------- | ----------------------------- |
| 1.1(1) | O(n)      | O(n)        | O(1)       | O(n)    | O(n)->O(1)    | 70, 198, 746, 790, 801        |
| 1.2(1) | O(n)      | O(n)        | O(n)       | O(n^2)  | O(n)          | 139, 818                      |
| 1.3(1) | O(n)+O(m) | O(mn)       | O(1)       | O(mn)   | O(mn)         | 72(712)                       |
| 1.4(2) | O(n)      | O(n^2)      | O(n)       | O(n^3)  | O(n^2)        | 312, 664, 673                 |
| 1.5(3) | O(n)      | O(n^3)      | O(n^3)     | O(n^4)  | O(n^3)        | 546                           |
| 1.6(1) | O(n), k   | O(k)        | O(n)       | O(kn)   | O(k)          | 322, 494                      |
| 1.7(2) | O(n), k   | O(kn)       | O(n)       | O(kn^2) | O(kn)         | 813                           |
| 2.1(1) | O(nm)     | O(nm)       | O(1)       | O(nm)   | O(nm)->O(m)   | 64(62, 63)                    |
| 2.2(2) | O(nm)     | O(kmn)      | O(1)       | O(kmn)  | O(kmn)->O(mn) | 688, Floyd-Warshall, 576, 741 |

- 列名
  - Case 模式，1 表示输入为 1 维，2 表示输入为 2 维，`(x)`表示难易度(数字越大越难)
  - Input 输入规模
  - Subproblems 子问题数
  - Depends on 每个子问题的复杂度
  - Time 时间复杂度
  - Space 空间复杂度，`->`表示可以优化
  - Problem IDS 在 LeetCode 中的序号，`()`表示非常相似的题

## 模板

### 1.1

```
Input O(n)
dp[i] := (opt-)sol of a subproblem (A[0->i])
dp[i] only depends on const smaller problems
Time: O(n)
Space: O(n)->O(1)
```

```py
Template

dp = [0] * n                        # init DP array
for i = 1 to n:                     # problem size
    dp[i] = f(dp[i-1], dp[i-2], ...)
return dp[n]
```

### 1.2

```
Input O(n)
dp[i] := (opt-)sol of a subproblem(A[0->i])
dp[i] depends on all smaller problems (dp[0], dp[1], ..., dp[i-1])
Time: O(n^2)
Space: O(n)
```

```py
Template

dp = new int[n]                             # init DP array
for i = 1 to n:                             # problem size
    for j = 1 to i - 1:                     # sub-problem size
        dp[i] = max/min(dp[i], f(dp[j]))    # find the opt sol from all opt sols of smaller problems
return dp[n]
```

```py
LC 139: Word break
dp[i] := wordBreak(A[0->i])
dp[i] = any(dp[i-k] && word(A[k+1->i])) // k 是分割点
```

```py
LC 818: Race Car
dp[i][0] := min steps to reach i facing right
dp[i][1] := min steps to reach i facing left

for k in range(1, i + 1):
    c = min(dp[k][0] + 2, dp[k][1] + 1)
    dp[i][0] = min(dp[i][0], dp[i - k][0] + c)
    dp[i][1] = min(dp[i][1], dp[i - k][1] + c)
```

### 1.3

```
Input O(m) + O(n), two arrays/strings
dp[i][j] := (opt-)sol of a subproblem (A[0->i][j], dp[i][j-1], dp[i-1][j-1])
Time: O(mn)
Space: O(mn)
```

```py
Template

dp = new int[n][m]      # init DP array
for i = 1 to n:         # problem size of input1
    for j = 1 to m:     # problem size of input2
        dp[i][j] = f(dp[i-1][j-1], dp[i-1][j], dp[i][j-1])
return dp[n][m]
```

### 1.4

```
Input O(n)
dp[i][j] := (opt-)sol of a subproblem (A[i->j]), a sub-array of the input
each subproblem depends on O(n) smaller problems
Time: O(n^3)
Space: O(n^2)
```

```py
Template

dp = new int[n][n]      # init a 2D array
for l = 1 to n:         # problem size
    for i = 1 to n:     # sub-problem start
        j = i + l - 1   # sub-problem end
        for k = i to j: # try all possible break points
            dp[i][j] = max(dp[i][j], f(dp[i][k], dp[k][j])) # merge two subproblems
return dp[1][n]
```

```py
LC 312. Burst Balloons
dp[i][j] := maxCoin(A[i->j])
dp[i][j] = max(dp[i][k-1] + dp[k+1][j] + C)
```

```py
LC 664. Strange Printer
dp[i][j] := min steps to print A[i->j]
dp[i][j] = min(dp[i][k] + dp[k+1][j-1])
```
