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
  - 中间状态需要某种容器保存(dp)，容器容量通常要大一些用来保存**初始状态**，同时也能避免边界条件逻辑复杂
  - 每一次状态转移都可以获得一次**局部最优解**
  - 局部最优解转移到下一步未必是**全局最优解**，但局部最优解一定是上一步局部最优解转移而来
  - 动态规划状态转移时的算法核心是**状态转移方程**，用来描述`dp[i]`是如何由前面的 dp 演化而来的
  - 有时要用二维数组`dp[i][j]`来存储中间值，最终可以只用一维数组，这时不要过早优化，完成之后再优化

- 获得 DP 算法的步骤：

  1. 发现迭代规律，从最简单开始构造 dp 数据结构(map/vector)（通常 dp array 需要比实际数量+1，为了 padding 补齐初始值），选定维度和方向(从简单问题逐步复杂的方向)。
  2. 设定一个状态转移方程，dp[i] = dp[i-1]...，先设定初始值，然后从 1(或 n-2)开始循环后面的元素。
  3. 最后演化出最终结果。*对于连续的最长值的动态规划的关键是识别打断连续时，之前的记录应该怎么起作用。*有时转移方程熟练之后可以发现并不需要存储大量内容就可以进行迭代。

- DP 一般可以由 dfs 转化而来，
  dfs 是从上到下的，而 dp 是从下到上（子问题结果逐步演化为最终结果）的，
  演化过程一般是：recur => recur + memory => dp array => dp。
  例：`740. Delete and Earn`、`72. Edit Distance`、`975. Odd Even Jump`
  - recur + memory
    可以将已计算的结果保存、重用。如果能控制 memory 中元素的"生长方向"则可以转换为"dp array"，也就是找到**状态转移方程**

### 思考： DP 和 DFS、BFS 的关系

- 既然 DP 可以由 DFS+Mem 推导而来，那是否可以用 DFS 替代 DP 呢？
  有些场景可以替代，但是有些场景演进方向不好控制，由于深度过深，很容易发生超出限时的问题
- 能否用 BFS 替代 DP?
  DP 的每次迭代其实和 BFS 的每次迭代类似，有些场景下可以通用。但是某些场景演进是要控制方向的，比如"Edit Distance"，要从横、纵两个方向也就是深度、广度两个方向控制演进，只能用 DP 了
- 如何从`DFS+二维Mem` 演化到 DP? 例如"312. Burst Balloons"
  可以画一个二维 Mem 的矩阵，然后在矩阵中用 DFS 演化一下过程，找到规律后，就可以变更为二维 DP 了。
  修改为 DP 后虽然时间复杂度相同，但是是避免了递归调用的性能损耗，整体性能可以明显提高

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

### 零一背包延伸——未必填满

- 对于有些题目，不一定能找到把背包完全填满的结果，但可以找到最优解。
  这时有两种方法取得最优解的最大容量：
  1. 如果在遍历过程中只记录了可以到达的状态，需要在最后反向遍历查找最大容量
  2. 可以直接在遍历过程中始终记录当前点可达到的最大值，这样直接取 DP 最后一个元素即可

参考：
"1049. Last Stone Weight II"

### 零一背包延伸——三维 DP

在零一背包基础上可以增加一个约束条件，需要增加一维 DP 解决。

参考：
"879. Profitable Schemes"

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

### LIS(Longest Increasing Subsequence)最大上升子序列

如：`300. Longest Increasing Subsequence`

- 每一步由过去所有 dp 结果遍历统计出
- 时间复杂度 O(n^2)

### 编辑距离问题(最小值)

例如`72. Edit Distance`

- 这类 DP 问题同样需要用二维数组迭代，迭代时要控制迭代方向，以便保证新一次迭代可由旧的已知结果导出

类似题目：
`1035. Uncrossed Lines`、`1143. Longest Common Subsequence`

### 区间 DP

目标是找到整个区间内的最值，遍历区间内的某个点作为分割点，求的最值，以分割点分割的两个子区间又可以继续向下分割。
区间 DP 可以用纯 DP 实现，但是用 DFS+Memory 模板更易理解记忆，性能差不太多。

- DFS+Memory 模板：

```Java
public int maxCoins(int[] nums) {
    int N = nums.length;
    Integer[][] memo = new Integer[N][N];
    return dfs(nums, 0, N-1, memo);
}
public int dfs(int[] nums, int start, int end, Integer[][] memo) {
    if (start > end) return 0;
    if (memo[start][end] != null) return memo[start][end];
    int max = Integer.MIN_VALUE;
    for (int i = start; i <= end; i++) {
        int left = dfs(nums, start, i-1, memo);
        int right = dfs(nums, i+1, end, memo);
        int cur = get(nums, i) * get(nums, start-1) * get(nums, end+1);
        max = Math.max(max, left + right + cur);
    }
    return memo[sstart][end] = max;
}
public int get(int[] nums, int i) {
    if (i == -1 || i == nums.length) return 1;
    return nums[i];
}
```

- 基本流程结构和 Minimax 模板类似，关键点是由递归子结果求得当前结果，利用 memo 剪枝
- 区间 DP 时，往往最两端需要添加虚拟元素，否则就需要像上面封装一个 get 函数。增加虚拟元素需要占用空间、封装 get 函数效率高一些。

示例：
"312. Burst Balloons"
"1039. Minimum Score Triangulation of Polygon"
"1547. Minimum Cost to Cut a Stick"

## 判断存在性

# 综合类型

## dp + bit

`338. Counting Bits`

# 动态规划的路径打印

## 记录关键值，最后反推

如："368. Largest Divisible Subset"

# 示例

### "72. Edit Distance"

```Golang
func minDistance(word1 string, word2 string) int {
    M, N := len(word1), len(word2)
    dp := make([][]int, M+1)
    for i := range dp {
        dp[i] = make([]int, N+1)
        dp[i][0] = i
    }
    for j := range dp[0] {
        dp[0][j] = j
    }
    for i := 1; i <= M; i++ {
        for j := 1; j <= N; j++ {
            dp[i][j] = dp[i-1][j-1]+1
            if word1[i-1] == word2[j-1] {
                dp[i][j] = dp[i-1][j-1]
            }
            dp[i][j] = min(dp[i][j], dp[i-1][j]+1)
            dp[i][j] = min(dp[i][j], dp[i][j-1]+1)
        }
    }
    return dp[M][N]
}
```

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

### "91. Decode Ways"

```Go
func numDecodings(s string) int {
    N := len(s)
    dp := make([]int, N+1)
    dp[0] = 1
    for i := 1; i <= N; i++ {
        if s[i-1] != '0' {
            dp[i] += dp[i-1]
        }
        if i > 1 && s[i-2] != '0' && ((s[i-2]-'0')*10+(s[i-1]-'0'))<=26 {
            dp[i] += dp[i-2]
        }
    }
    return dp[N]
}
```

```Go
func numDecodings(s string) int {
    N := len(s)
    dp1, dp2, dp3 := 0, 1, 0
    for i := 1; i <= N; i++ {
        dp3 = 0
        if s[i-1] != '0' {
            dp3 += dp2
        }
        if i > 1 && s[i-2] != '0' && ((s[i-2]-'0')*10+(s[i-1]-'0'))<=26 {
            dp3 += dp1
        }
        dp1, dp2 = dp2, dp3
    }
    return dp3
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

### "131. Palindrome Partitioning" Golang

```Go
func partition(s string) [][]string {
    N := len(s)
    dp := make([][]bool, N)
    for i := range dp {
        dp[i] = make([]bool, N)
        for j := range dp[i] {
            dp[i][j] = true
        }
    }
    for i := N-1; i >=0; i-- {
        for j := i+1; j < N; j++ {
            dp[i][j] = s[i]==s[j] && dp[i+1][j-1]
        }
    }

    var ret [][]string
    var splits []string
    var dfs func(int)
    dfs = func(i int) {
        if i == N {
            ret = append(ret, append([]string(nil), splits...))
        }
        for j := i; j < N; j++ {
            if dp[i][j] {
                splits = append(splits, s[i:j+1])
                dfs(j+1)
                splits = splits[:len(splits)-1]
            }
        }
    }
    dfs(0)
    return ret
}
```

### "132. Palindrome Partitioning II" Golang

```Go
func minCut(s string) int {
    N := len(s)
    dp1 := make([][]bool, N)
    for i := range dp1 {
        dp1[i] = make([]bool, N)
        for j := range dp1[i] {
            dp1[i][j] = true
        }
    }
    for i := N-1; i >= 0; i-- {
        for j := i+1; j < N; j++ {
            dp1[i][j] = s[i]==s[j] && dp1[i+1][j-1]
        }
    }

    dp2 := make([]int, N)
    for i := range dp2 {
        if dp1[0][i] {
            continue
        }
        dp2[i] = math.MaxInt64
        for j := 0; j < i; j++ {
            if dp1[j+1][i] && dp2[j]+1 < dp2[i] {
                dp2[i] = dp2[j]+1
            }
        }
    }
    return dp2[N-1]
}
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

### "198. House Robber"

```Go
func rob(nums []int) int {
    if len(nums) == 1 {
        return nums[0]
    }
    first := nums[0]
    second := max(first, nums[1])
    for i := 2; i < len(nums); i++ {
    // 推演过程：
    //    do := first + nums[i] // 抢劫当前
    //    notDo := max(first, second)   // 不抢劫当前
    //    first, second = notDo, do
        first, second = max(first, second), first + nums[i]
    }
    return max(first, second)
}
```

### "213. House Robber II"

```Go
func rob(nums []int) int {
    if len(nums) == 1 {
        return nums[0]
    }
    return max(subRob(nums[:len(nums)-1]), subRob(nums[1:]))
}
func subRob(nums []int) int {
    if len(nums) == 1 {
        return nums[0]
    }
    first, second := nums[0], max(nums[0], nums[1])
    for i := 2; i < len(nums); i++ {
        first, second = second, max(second, first+nums[i])
    }
    return max(first, second)
}
```

### 279. Perfect Squares

```Go
func numSquares(n int) int {
    dp := make([]int, n+1)
    for i := 1; i <= n; i++ {
        minVal := math.MaxInt32
        for j := 1; j*j <= i; j++ {
            minVal = min(minVal, dp[i-j*j])
        }
        dp[i] = minVal+1
    }

    return dp[n]
}
```

### "300. Longest Increasing Subsequence"

```Go
func lengthOfLIS(nums []int) (ret int) {
    dp, ret := make([]int, len(nums)), 0
    for i := 0; i < len(nums); i++ {
        for j := 0; j < i; j++ {
            if nums[i] > nums[j] {
                dp[i] = max(dp[i], dp[j]+1)
                ret = max(ret, dp[i])
            }
        }
    }
    return ret+1
}
```

### "312. Burst Balloons"

```Go
// DFS + Memory
func maxCoins(nums []int) int {
    n := len(nums)
    getCoin := func(idx int) int {
        if idx < 0 || idx == n {
            return 1
        }
        return nums[idx]
    }

    memo := make([][]int, n)
    for i := range memo {
        memo[i] = make([]int, n)
    }

    var dfs func(l, r int) int
    dfs = func(l, r int) int {
        if l > r { return 0 }
        if memo[l][r] != 0 { return memo[l][r] }
        ret := int(math.MinInt32)
        for k := l; k <= r; k++ {
            left := dfs(l, k-1)
            right := dfs(k+1, r)
            cur := getCoin(k) * getCoin(l-1) * getCoin(r+1)
            ret = max(ret, cur + left + right)
        }
        memo[l][r] = ret
        return ret
    }

    return dfs(0, n-1)
}
```

```Go
// DP
func maxCoins(nums []int) int {
    N := len(nums)
    vals := make([]int, N+2)
    vals[0], vals[N+1] = 1, 1
    for i := range nums {
        vals[i+1] = nums[i]
    }
    dp := make([][]int, N+2)
    for d := range dp {
        dp[d] = make([]int, N+2)
    }
    for l := 1; l <= N; l++ {
        for i := 1; i+l < N+2; i++ {
            j := i + l - 1
            for k := i; k <= j; k++ {
                dp[i][j] = max(dp[i][j], dp[i][k-1] + vals[i-1]*vals[k]*vals[j+1] + dp[k+1][j])
            }
        }
    }
    return dp[1][N]
}
```

### "338. Counting Bits" Golang

```Go
func countBits(num int) []int {
    dp := make([]int, num+1)
    for i := 1; i <= num; i++ {
        dp[i] = dp[i&(i-1)]+1
    }
    return dp
}
```

### "354. Russian Doll Envelopes" Golang

```Go
func maxEnvelopes(envelopes [][]int) int {
    if len(envelopes) == 0 {
        return 0
    }

    sort.Slice(envelopes, func(a, b int) bool {
        return envelopes[a][0] < envelopes[b][0] || (envelopes[a][0] == envelopes[b][0] && envelopes[a][1] > envelopes[b][1])
    })

    dp := make([]int, len(envelopes))
    for i := 0; i < len(envelopes); i++ {
        for j := 0; j < i; j++ {
            if envelopes[i][1] > envelopes[j][1] {
                dp[i] = max(dp[i], dp[j]+1)
            }
        }
    }

    return max(dp...)+1
}

func max(arr ...int) int {
    ret := ^int(^uint(0)>>1)    // set to MIN_INT
    for _, v := range arr {
        if v > ret {
            ret = v
        }
    }
    return ret
}
```

### "368. Largest Divisible Subset"

```Go
func largestDivisibleSubset(nums []int) (ret []int) {
    sort.Ints(nums)
    dp := make([]int, len(nums))
    for i := range dp {
        dp[i] = 1
    }

    maxVal, maxLen := 1, 1
    for i, n := range nums {
        for j, v := range nums[:i] {
            if n%v == 0 && dp[i] < dp[j]+1 {
                dp[i] = dp[j]+1
            }
        }
        if dp[i] > maxLen {
            maxLen, maxVal = dp[i], nums[i]
        }
    }

    if maxLen == 1 {
        return []int{nums[0]}
    }

    for i := len(nums)-1; i >= 0; i-- {
        if dp[i] == maxLen && maxVal%nums[i] == 0 {
            ret = append(ret, nums[i])
            maxLen, maxVal = maxLen-1, nums[i]
        }
    }
    return ret
}
```

### "377. Combination Sum IV" Golang

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
    sum := 0
    for _, n := range nums {
        sum += n
    }
    if sum & 1 == 1 {
        return false
    }
    tar := sum>>1
    dp := make([]bool, tar+1)
    dp[0] = true

    for _, n := range nums {
        for i := tar; i > 0 && (i-n) >= 0; i-- {
            dp[i] = dp[i] || dp[i-n]
        }
    }

    return dp[tar]
}
```

### "474. Ones and Zeroes"

```Go
func findMaxForm(strs []string, m int, n int) int {
    dp := make([][]int, m+1)
    for i := range dp {
        dp[i] = make([]int, n+1)
    }
    for _, s := range strs {
        zeros := strings.Count(s, "0")
        ones := len(s) - zeros
        for i := m; i >= zeros; i-- {
            for j := n; j >= ones; j-- {
                dp[i][j] = max(dp[i][j], dp[i-zeros][j-ones] + 1)
            }
        }
    }
    return dp[m][n]
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

### "664. Strange Printer" Golang

```Go
func strangePrinter(s string) int {
    N := len(s)
    dp := make([][]int, N)
    for i := range dp {
        dp[i] = make([]int, N)
    }
    var dfs func(string, int, int) int
    dfs = func(s string, i, j int) int {
        if i > j {
            return 0
        } else if dp[i][j] > 0 {
            return dp[i][j]
        }

        dp[i][j] = 1 + dfs(s, i, j-1)
        for k := i; k < j; k++ {
            if s[k] == s[j] {
                // k 和 j 位置的打印次数算在前面，后面可以少算这两个字符
                dp[i][j] = min(dp[i][j], dfs(s, i, k) + dfs(s, k+1, j-1))
            }
        }
        return dp[i][j]
    }
    return dfs(s, 0, N-1)
}
```

### "818. Race Car"

```Go
func racecar(target int) int {
    dp := make([]int, target+3)
    dp[0], dp[1], dp[2] = 0, 1, 4
    for t := 3; t <= target; t++ {
        dp[t] = math.MaxInt32
        k := 32 - bits.LeadingZeros32(uint32(t))
        if (t == (1<<k)-1) {
            dp[t] = k
            continue
        }
        for j := 0; j < k-1; j++ {
            dp[t] = min(dp[t], dp[t-(1<<(k-1))+(1<<j)]+k-1+j+2)
        }
        if (1<<k) - 1 - t < t {
            dp[t] = min(dp[t], dp[(1<<k)-1-t]+k+1)
        }
    }
    return dp[target]
}

func min(a, b int) int {
    if a < b {
        return a
    }
    return b
}
```

### "879. Profitable Schemes"

```Go
func profitableSchemes(n int, minProfit int, group []int, profit []int) (ret int) {
    const MOD = 1e9 + 7
    groupLen := len(group)  // 工作(物品)数量，用于主循环
    dp := make([][][]int, groupLen+1)   // 工作、人数、最小获利
    for i := range dp {
        dp[i] = make([][]int, n+1)
        for j := range dp[i] {
            dp[i][j] = make([]int, minProfit+1)
        }
    }
    dp[0][0][0] = 1
    for i, num := range group { // 这里下标偏小，后面要+1
        for j := 0; j <= n; j++ {
            for k := 0; k <= minProfit; k++ {
                if j < num {    // 人数不满足当前工作，延续前面的结果
                    dp[i+1][j][k] = dp[i][j][k]
                } else {        // 人数可以满足当前工作
                    dp[i+1][j][k] = (dp[i][j][k] + dp[i][j-num][max(0, k-profit[i])]) % MOD
                }
            }
        }
    }

    for _, d := range dp[groupLen] {  // 遍历累加全部物品、不同人数、满足 minProfit 的结果
        ret = (ret + d[minProfit]) % MOD
    }
    return ret
}
```

```Go
// 代码简化
func profitableSchemes(n int, minProfit int, group []int, profit []int) (ret int) {
    const MOD = 1e9 + 7
    dp := make([][]int, n+1)   // 人数、最小获利
    for i := range dp {
        dp[i] = make([]int, minProfit+1)
        dp[i][0] = 1    // 赋予可能性
    }
    for i, num := range group { // 工作(物品)
        for j := n; j >= 0; j-- {   // 逆序，避免后效
            for k := minProfit; k >= 0 ; k-- {  // 逆序，避免后效
                if j >= num {        // 人数可以满足当前工作
                    dp[j][k] = (dp[j][k] + dp[j-num][max(0, k-profit[i])]) % MOD
                }
            }
        }
    }
    return dp[n][minProfit]
}
```

### "983. Minimum Cost For Tickets"

```Go
// dfs + memo
func mincostTickets(days []int, costs []int) int {
    memo := make([]int, 366)
    dayMap := make(map[int]bool, len(days))
    for _, d := range days {
        dayMap[d] = true
    }

    var dfs func(int) int
    dfs = func(day int) int {
        if day > 365 {
            return 0
        }
        if memo[day] > 0 {
            return memo[day]
        }
        if dayMap[day] {
            memo[day] = min(dfs(day+1)+costs[0], dfs(day+7)+costs[1], dfs(day+30)+costs[2])
        } else {
            memo[day] = dfs(day+1)
        }
        return memo[day]
    }
    return dfs(1)
}
```

```Go
// dp
func mincostTickets(days []int, costs []int) int {
    dp := make([]int, 365+30+1)
    idx := len(days)-1
    for i := 365; i > 0; i-- {
        if idx >= 0 && i == days[idx] {
            dp[i] = min(dp[i+1]+costs[0], dp[i+7]+costs[1], dp[i+30]+costs[2])
            idx--
        } else {
            dp[i] = dp[i+1]
        }
    }
    return dp[1]
}
```

### "1049. Last Stone Weight II"

```Go
// 最后遍历
func lastStoneWeightII(stones []int) int {
    sum := 0
    for _, v := range stones {
        sum += v
    }
    m := sum / 2
    dp := make([]bool, m+1)
    dp[0] = true
    for _, weight := range stones {
        for j := m; j >= weight; j-- {
            dp[j] = dp[j] || dp[j-weight]
        }
    }
    for j := m; ; j-- {
        if dp[j] {
            return sum - 2*j
        }
    }
}
```

```Go
// 记录最大值
func lastStoneWeightII(stones []int) int {
    sum := 0
    for _, v := range stones {
        sum += v
    }
    m := sum >> 1
    dp := make([]int, m+1)
    for _, weight := range stones {
        for j := m; j >= weight; j-- {
            dp[j] = max(dp[j], dp[j-weight]+weight)
        }
    }
    return sum - dp[m]*2
}
```

### "1143. Longest Common Subsequence"

```Go
func longestCommonSubsequence(text1 string, text2 string) int {
    m, n := len(text1), len(text2)
    dp := make([][]int, m+1)
    for i := range dp {
        dp[i] = make([]int, n+1)
    }
    for i, v1 := range text1 {
        for j, v2 := range text2 {
            if v1 == v2 {
                dp[i+1][j+1] = dp[i][j]+1
            } else {
                dp[i+1][j+1] = max(dp[i][j+1], dp[i+1][j])
            }
        }
    }
    return dp[m][n]
}
```

### "1262. Greatest Sum Divisible by Three"

```Go
func maxSumDivThree(nums []int) int {
    dp := make([]int, 3)    // 不同余数最大和
    for _, n := range nums {
        a, b, c := dp[0]+n, dp[1]+n, dp[2]+n
        iA, iB, iC := a%3, b%3, c%3
        // 必须分别更新，否则可能相互影响
        dp[iA] = max(dp[iA], a)
        dp[iB] = max(dp[iB], b)
        dp[iC] = max(dp[iC], c)
    }
    return dp[0]
}
```

### "1547. Minimum Cost to Cut a Stick"

```Go
func minCost(n int, cuts []int) int {
    N := len(cuts)
    sort.Ints(cuts)
    memo := make([][]int, N+1)
    for i := range memo {
        memo[i] = make([]int, N+1)
    }

    var dfs func(l, r int) int
    dfs = func(l, r int) int {
        if l >= r {
            return 0
        } else if memo[l][r] != 0 {
            return memo[l][r]
        }
        rightPos, leftPos := n, 0
        if l > 0 {
            leftPos = cuts[l-1]
        }
        if r < N {
            rightPos = cuts[r]
        }
        cost := rightPos - leftPos
        memo[l][r] = math.MaxInt32
        for i := l; i < r; i++ {
            memo[l][r] = min(memo[l][r], dfs(l, i) + cost + dfs(i+1, r))
        }
        return memo[l][r]
    }

    return dfs(0, N)
}
```
