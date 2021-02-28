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

- 如何识别是动态规划题？下面字面可以作为参考条件：

  - 1. 计数
    - 有多少种方式走到右下角
    - 有多少种方法选出 k 个数使得和是 sum
  - 2. 求最大最小值
    - 从左上角走到右下角路径的最大数字和
    - 最长上升子序列长度
  - 3. 求存在性
    - 取石子游戏，先手是否必胜
    - 能不能选出 k 个数使得和是 sum

- LeetCode 中常见动态规划类型及其比例
  - 坐标型动态规划(20%)
  - 序列型动态规划(20%)
  - 划分型动态规划(20%)
  - 区间型动态规划(15%)
  - 背包型动态规划(10%)
  - 最长序列型动态规划(5%)
  - 博弈型动态规划(5%)
  - 综合型动态规划(5%)

## 基本算法

- 思路：

  - 最终结果是由**一系列状态转移逐步得出**的，状态转移是有**顺序**的，后面的状态可能性可由前面的状态推出
  - 中间状态需要某种容器保存(dp)，容器容量通常要大一些用来保存**初始状态**
  - 每一次状态转移都可以获得一次**局部最优解**
  - 局部最优解转移到下一步未必是**全局最优解**，但局部最优解一定是上一步局部最优解转移而来
  - 动态规划状态转移时的算法核心是**状态转移方程**，用来描述`dp[i]`是如何由前面的 dp 演化而来的
  - 有时要用二维数组`dp[i][j]`来存储中间值，最终可以只用一维数组，这时不要过早优化，完成之后再优化

- 获得 DP 算法的步骤：
  1. 发现迭代规律，从最简单开始构造 dp 数据结构(map/vector)（通常 dp array 需要比实际数量+1，为了 padding 补齐初始值），选定维度和方向(从简单问题逐步复杂的方向)。
  2. 设定一个状态转移方程，dp[i] = dp[i-1]...，先设定初始值，然后从 1(或 n-2)开始循环后面的元素。
  3. 最后演化出最终结果。*对于连续的最长值的动态规划的关键是识别打断连续时，之前的记录应该怎么起作用。*有时转移方程熟练之后可以发现并不需要存储大量内容就可以进行迭代。
  - DP 一般可以由 dfs 转化而来，dfs 是从上到下的，而 dp 是从下到上（子问题结果逐步演化为最终结果）的，演化过程一般是：recur => recur + memory => dp array => dp。例：`740. Delete and Earn`、`72. Edit Distance`、`975. Odd Even Jump`

### 思考： DP 和 DFS、BFS 的关系

- 既然 DP 可以由 DFS+Mem 推导而来，那是否可以用 DFS 替代 DP 呢？
  有些场景可以替代，但是有些场景演进方向不好控制，由于深度过深，很容易发生超出限时的问题
- 能否用 BFS 替代 DP?
  DP 的每次迭代其实和 BFS 的每次迭代类似，有些场景下可以通用。但是某些场景演进是要控制方向的，比如"Edit Distance"，要从横、纵两个方向也就是深度、广度两个方向控制演进，只能用 DP 了

## 零一背包问题

- 题目：
  有一个背包体积是 V，有 n 个物品体积为`v[i]`价值为`p[i]`，问如何把最大价值的物品放入背包
- 思路：
  - 在**零一背包**问题中选取某物品后就不能再选，这就是**后效性**，是指做了某种选择后会影响后面的状态决策。
  - 动态规划演进过程中因为无法判断当前枚举的状态的来源，所以不允许出现后效性，如果解决不了则不能使用动态规划。
  - 由于**后效性**会导致状态迁移发生逻辑冲突，比如"拿过的物品下次还会再被选择"，所以我们要想办法消除后效性。
    解决方法需要保证两个顺序：
    1. **先遍历决策再遍历状态**保证每个决策只在每个状态上最多应用一次，比如"一个物品只拿一次"
    2. 如果同层状态会影响"后面的"状态，则利用**倒序**消除这种影响
  - 无**后效性**的例子比如"用最少数量的硬币凑够目标值"，用过的硬币不影响以后再用。不过同样要保证演进方向，这是 dp 算法的基础。

## 完全背包

完全背包是把零一背包中"每个物品只有一个"的限制去掉，这样算法其实更容易了。
基本算法和零一背包一致，不同的地方是在同层处理时，不需要考虑物品自己对自己的影响(无**后效性**)，所以可以去掉"倒序"的处理逻辑。

### 优化：提前去掉低性价比物品

在相同体积下，可能有不同的价值的物品。这时可以提前选取其中"体(积)价(格)比"最低的作为待选物品。

## 多重背包

多重背包规定每种物品的数量是有限但不唯一。相比零一背包，多重背包更难一些。
比较简单的思路，是把多重背包中同一个种类的物品拆开，这样就可以看作零一背包来处理，但是由于物品数量可能很大，性能可能是无法接受的。

### 优化：二进制表示法

如果某种物品的数量特别巨大，比如`1e9`，这时要把这种物品拆解为单个物品再套用零一背包的方法，因为数量太大而不可行。

为什么一个很大的 int 数，只要 32 位就可以保存呢？
因为其中某一位就可以表示`比它低一位的数值*2`，这样我们只要枚举组合最多 32 次，就能描述出一个具体的数。
比如`5 == b101 == b100 + b1`，只需要枚举两次而不用`1+1+1+1+1`枚举 5 次
应用上面思路，我们把某种物品的数量分解成最多 32 个包，每个包的值就是其中元素的总和值，然后对这些值进行枚举相加，就可以组合出所有可能的数量

- 操作流程：
  1. 按照数量从低(`1`)到高(`1<<31`)，逐步计算物品数量可满足的量级，把数量(包的值)值存入数组
  2. 保存最多 32 个数(往往没有这么多)，最后可能有余数，没关系余数也算做一个包即可
  3. 把这些分好的包当作唯一物品，做零一背包处理即可

```Go
// 二进制拆分，输入参数：总数、单个体积、单个价值，输出参数：数组元素是包体积、价值
func BinaryDivide(cnt, vol, pri int) (ret [][]int) {
    for i := 0; i < 32; i++ {
        cur := 1 << i
        if cnt < cur {
            ret = append(ret, []int{cnt*vol, cnt*pri})
            break
        } else {
            cnt -= cur
            ret = append(ret, []int{cur*vol, cur*pri})
        }
    }
    return ret
}
```

### 优化：单调队列

假设包体积 V、物品个数 N、
二进制优化后，时间复杂度被大大缩短，但是仍然为 O()

## 概率/(组合)次数统计问题

还有一种 DP 问题，和零一背包类似，不过策略复杂一些，对同一个物品可能进行多种操作(例如象棋有多个可能摆放位置)。
这种问题往往最终要求是某个状态的概率或总可能次数。例如`494. Target Sum`

- 这种问题一般通过二维数组演进，通过逐行演进，避免同一物品不同策略对自己的**后效性**
- 第一层对物品循环、第二层对状态循环、第三层对同一物品的不同策略循环
- 在这种问题中，通常状态是一个**有限范围**(比如棋盘范围)，这样在一个范围内可以逐代累加统计结果
- 用二维数组完成后，会发现其实每次迭代一个物品只与上一个物品迭代结果有关，则可以只保留两个数组迭代，降低空间复杂度

### 不计组合顺序(完全背包)

例如`518. Coin Change 2`
如果物品可以不限次数使用、不计顺序，则只要两层循环，并且第二层循环可以正序执行。类比完全背包算法

### 统计组合顺序

例如`377. Combination Sum IV`
第一层循环状态、第二层循环物品

## 最大最小值

### 编辑距离问题(最小值)

例如`72. Edit Distance`

- 这类 DP 问题同样需要用二维数组迭代，迭代时要控制迭代方向，以便保证新一次迭代可由旧的已知结果导出

## 判断存在性

## 动态规划的路径打印

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

### "377. Combination Sum IV" Golang DP

```Go
func combinationSum4(nums []int, target int) int {
    dp := make([]int, target + 1)
    for i := 1; i <= target; i++ {
        for _, j := range nums {
            if i == j {
                dp[i] += 1
                continue
            }
            if i > j && dp[i-j] != 0 {
                dp[i] += dp[i-j]
            }
        }
    }
    return dp[target]
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

### "518. Coin Change 2" Golang

```Go
func change(amount int, coins []int) int {
    dp := make([]int, amount+1)
    dp[0] = 1
    for _, coin := range coins {
        for i := 0; i <= amount; i++ {
            if i - coin >= 0 && dp[i-coin] > 0 {
                dp[i] += dp[i-coin]
            }
        }
    }
    return dp[amount]
}
```

### "123. Best Time to Buy and Sell Stock III" Golang

```Go
func maxProfit(prices []int) int {
    buy1, sell1, buy2, sell2 := -prices[0], 0, -prices[0], 0
    for i := 1; i < len(prices); i++ {
        buy1 = max(buy1, -prices[i])
        sell1 = max(sell1, buy1 + prices[i])
        buy2 = max(buy2, sell1 - prices[i])
        sell2 = max(sell2, buy2 + prices[i])
    }
    return sell2
}
func max(a, b int) int {
    if a >= b {
        return a
    }
    return b
}
```
