---
layout: post
title: "LeetCode Question Category"
date: 2020-12-27 08:00:00 +0800
tags: Algorithm Leetcode
---

![LeetCode](/assets/images/2020-12-27-LeetCode_Category_1.jpg)
记录 LeetCode 题型分类及解题思路

### "11. Container With Most Water"

- 解法 1(推荐)：

  - 高度、距离都是影响水量的值，其中高度的权重更大，所以在循环缩小距离时保持较高的高度让高度较小的一方遍历。

- 解法 2：
  - 利用动态规划法，分别计算当前位置左边最大高度、右边最大高度，算法较易理解，但是需要占用额外空间。

### "31. Next Permutation"

- 原理：
  顺序排列的数组没有更小的值，倒序排列的数组没有更大的值。
- 解法：
  1. 从右向左查找第一个发生递减的位置。
  2. 从此位置向右查找最右边比当前值高一点的位置。
  3. 交换这两个位置。
  4. 将发生递减位置右边所有位置反转，呈现正序排列。

["31. Next Permutation" Array]()

### "32. Longest Valid Parentheses"

- 思路 1：stack 背影法
  在 stack 中保存有效和无效位置，默认填-1。如果中间持续是有效位置，那么到有效匹配后，就可以看一下之前的背影最远到哪里，记录这个最大值。

["32. Longest Valid Parentheses" Stack]()

- 思路 2: 双向遍历
  - 第一遍从左向右，用 left、right 分别记录左右括号数量，如果`left == right`则记录最大值
  - 为了方式忽略`(()`，从右向左再遍历一遍，取两次遍历最大值为最终结果
- 关键点：
  - 遍历过程中，如果应该后出现的符号过多(比如从左到右时`)`过多)，则要重置计数

["32. Longest Valid Parentheses" Brute Force]()

### "49. Group Anagrams"

- 思路：
  利用 Anagram(异位词)特性，统计每个词的字母出现次数，生成一个"指纹"，然后通过 HashMap 收集相同指纹的词

["49. Group Anagrams" HashTable]()

### "33. Search in Rotated Sorted Array"

- 思路：
  这道题是在二分查找的基础上，增加一层排除法逻辑，使得每次都能缩小一半查找范围。

["33. Search in Rotated Sorted Array" Binary Search]()

### "34. Find First and Last Position of Element in Sorted Array"

- 思路：
  这道题是在二分查找的基础上，进一步利用算法细节特性："二分查找的结果是元素第一次出现的下标"。
  所以只需要再查询`按排序下一个元素出现的位置-1`就可以得到末尾位置

["34. Find First and Last Position of Element in Sorted Array" Golang]()

### "42. Trapping Rain Water"

- 思路 1:两遍遍历
  假设每一个下标对应一个水柱，则水柱的高度由此位置最左和最右 bar 的高度综合决定。

  1. 从左向右遍历一遍，记录每个下标对应的左边最高的 bar 高度
  2. 从右向左遍历一遍，一边遍历一边计算累加当前位置的水柱高度，`水柱高度 = min(left, right) * height`

  - 时间复杂度 O(n)，空间复杂度 O(n)

- 思路 2:双指针
  按思路 1 计算每个水柱时，高度取决于较低的那个边。所以可以单独对高度较低的边进行遍历，这样只需要遍历一遍，并且无需额外空间

["42. Trapping Rain Water" TwoPointers Golang]()

### "64. Minimum Path Sum"

- 思路：这道题有较好的 DP 解决方案，所以用 DFS 时性能相比较差就会超时。解决思路：从右下角开始动态规划，计算每个下一个位置可能的最小值。利用原空间就可以完整 DP 过程。

### "72. Edit Distance"

- 编辑距离在机器翻译、语音识别等数据科学领域经常用到。
- 动态规划解法：
  1. 用 dp[i][j]代表 word1[i]到 word2[j]的编辑距离，从空字符串开始演化，所以二维矩阵的范围要大 1。
  2. 初始化第 0 行和第 0 列，然后遍历计算 dp[i][j]。转移公式是：dp[i][j] = min(dp[i-1][j]+1, min(dp[i][j-1]+1, dp[i-1][j-1] + (word1[i] == word2[j]? 0, 1)))。
  3. 最后返回 dp[m][n]。

["72. Edit Distance" DP Golang]()

### "73. Set Matrix Zeroes"

- 考点：
  - "in place"思想
  - 二进制压缩，用单独标记记录第一行和第一列
  - 利用逆序处理，利用第一行的标记作用

["73. Set Matrix Zeroes" Array C++]()

### "78. Subsets"

- 迭代法：
  可以将迭代过程看做：`[[]] -> [[],[1]] -> [[],[1],[2],[1,2]] -> [[],[1],[2],[1,2],[3],[1,3],[2,3],[1,2,3]]`
  每次将前面的内容复制到结尾，并将新的数字加入进去。

["78. Subsets" C++]()

- 回朔法：
  设置一个集合切片，每一个元素可以选取或不选取

["78. Subsets" Golang]()

### "84. Largest Rectangle in Histogram"

- 这道题的**单调栈**解题思路（时间复杂度 O(n)）很重要，在很多其他题中都会用到。
- 基本原理：
  1. 从左向右遍历，并不断将高度值的下标 push 入 stack。
  2. 当某一位置 height 值缩小时，那么之后就很难计算前面的面积了，所以就要把前面比当前值高的 stack 出栈，并计算前面的面积。
  3. 注意，计算面积时，宽度要当前下标和前面的"背影"相减，所以栈底是-1。

["84. Largest Rectangle in Histogram" Golang]()

### "123. Best Time to Buy and Sell Stock III"

- 考点：
  - DP
  - 临界条件逻辑
- 思路：
  - 对于任意一个结束日 i，有 5 个**利润状态值**需要从 i-1 的状态推导，这五个状态值及其动态转移方程为：
    - 假设前一日的结果标记为`'`，买卖产生的利润为`price[i]-price'`
    1. 无操作利润，始终为 0，无需状态迁移
    2. buy1 当日第一次买入，max(buy1', -price[i])
    3. sell1 当日第一次卖出，max(sell1', buy1' + price[i])
    4. buy2 当日第二次买入，max(buy2', sell1' - price[i])
    5. sell2 当日第二次卖出，max(sell2', buy2' + price[i])
  - 如果新一日价格更低，则`-price[i] > -price'`，所以上面 buy1 的公式符合逻辑
  - 对于临界条件，如果同一天做完上述四个动作，则 sell1、sell2 的利润都为 0。我们可以直接利用当天的数据作为迭代初始数据而不会影响结果
  - 又根据上面推导，如果只进行一次交易，sell2 的值和 sell1 一致，所以我们最后直接返回 sell2 就可以
  - 只要根据第一天数据设置初始值，后面继续迭代就能推导出最后结果

["123. Best Time to Buy and Sell Stock III" Golang]()

### "131. Palindrome Partitioning"

- 解法：backtracking + dp
  - 设 dp[i][j] 为 "i~j 是否为回文子串"，先用 dp 的方法预处理，求出所有连续字串是否为回文字串
  - 用回溯的方法尝试从 i~j 进行分割，然后 dfs 剩下的字符串
- 思路：
  - dp 的初始状态是所有值均为 true，然后逐步计算出结果，`dp[i][j] = s[i]==s[j] && dp[i+1][j-1]`
  - dp 的 i、j 两层循环方向很重要，为了符合`i<j`的语义，要控制好遍历方向
  - backtracking 时要注意做好数组的复制，避免回溯删除动作影响了最终结果

["131. Palindrome Partitioning" Golang]()

### "132. Palindrome Partitioning II"

- 解法：两次 dp
  - 参照`131`可以生成一个辅助 dp1，可以快速求得 dp1[i][j]是否符合回文
  - dp2[j] 表示到 j 为止最少的分割次数，设 i 为一个分割点(0 <= i <= j)使得 i+1~j 是回文，即`dp1[i+1][j] == true`，`dp2[j] = min(dp2[j], dp2[i]+1)`

["132. Palindrome Partitioning II" Golang]()

### "139. Word Break"

- 思路：dp
  - 这是一个完全背包问题
  - dp[i]表示 i 长度的字符串能否用字典表示，0 < i <= stringLength
  - dp[i]演化过程中，只需要判别末尾是否能匹配一个单词

["139. Word Break" Golang]()

### "141. Linked List Cycle"

- 解法：快慢指针法
  1. 从起始出发出两个指针，一个"快指针"和一个"慢指针"
  2. 快指针每次跳两个位置、慢指针每次跳一个位置
  3. 如果不存在回环(loop)，则快指针会先到 null 节点
  4. 如果存在回环，则快慢指针必然在某点会相遇
- 此解法时间复杂度为 O(n)、空间复杂度为 O(1)

### "142. Linked List Cycle II"

- 解法"Floyd's Tortoise and Hare"算法。
  1. 通过快慢指针判断是否存在回环，并且记录快慢指针相交位置。
  2. 从链表头部和之前的快慢指针相交位置同时发起一个慢指针循环继续向前
  3. 当两指针重叠时，就找到了回环进入点

["142. Linked List Cycle II" Golang]()

### "146. LRU Cache"

- 设计：
  - 用 LinkedList 记录元素被更新的时间顺序，被用到时就放在链表最后，链表元素要记录 Key-Value 值
  - 用 HashMap 记录元素是否存在，其 Value 保存`*ListElement`，用于 O(1)快速定位元素
  - 在 size 发生变化时，如果 size 超过容量，则删除 list 头元素

["146. LRU Cache" Golang]()

### "152. Maximum Product Subarray"

- 思路：
  由于存在负数元素，之前计算的最大值可能会一下反转为最小值；相反，之前计算的最小负值可能一下变成最大值。
  所以在采用动态规划法时，要保留最大、最小两个值，然后向后演进。

### "155. Min Stack"

- 思路：
  stack 的特性是"旧数据保持不变"
- 设计：
  增加一个"辅助 stack"，用于保存到目前为止的最小值

### "160. Intersection of Two Linked Lists"

- 解法：
  1. 分别从 A、B 两个链表的起点开始一个跳转指针
  2. 当指针到达末尾后，切换两个指针的位置为 B、A，继续跳转
  3. 如果两个指针指向同一位置，则该位置就是链表交汇点
  4. 如果任意指针走到了尾部仍然没有交汇，则没有交汇点
- 思路：
  这种问题先简化思路，假设有两个链表，分别为:`A->C->D`和`B->C->D`，则 C 是交汇点。如果按照上面方法使跳转路径相交叠，则两个指针交汇前走的距离分别是：`AC+CD+BC`和`BC+CD+AB`，可以看到，这两个距离是完全相等的
- 这个解法的优点是空间复杂度为 O(1)

### "169. Majority Element"

- 思路：
  这类求**Majority(众数)**的问题，可以用**Boyer-Moore(摩尔投票法)**解决，基本思路就是"小弟够多不怕对砍"。
- 步骤：
  1. 对某个数进行计数并记录该数，如果遇到这个数则计数加一;
  2. 如果不是这个数则计数减一，如果为零则重新计数。
  3. 最后如果计数大于 1，则遍历数组对找到的数计数，如果总数超过了 1/2，则找到了目标。

["169. Majority Element]()

### "229. Majority Element II"

- 思路：
  **众数问题**用**摩尔投票法**
- 不同点：
  1. 统计的数成了两个
  2. 最后统计时，最终结果元素要超过统计结果的 1/3

### "238. Product of Array Except Self"

- 题目要求：
  1. 不允许用除法。
  2. 空间复杂度为 O(1)，(输出数组不算在内)。
- 解决方案：
  1. 创建输出数组，遍历原数组，将左边的乘积记录入输出数组。
  2. 从右向左遍历原数组，累积将右边的乘积计算到输出数组。

### "239. Sliding Window Maximum"

- 解法：
  这道题要用到 DQ(Double Queue)，在 Window 逐步移动过程中，通过 DQ 维护一个最大值队列的下标
  1. 检查 DQ 末尾，如果对应的值比当前值更小，则弹出，最终把当前值插入。其本质是利用后端的栈结构维护一个最有效的下标值队列
  2. 检查 DQ 头部，如果对应的下标值已经超出了有效范围，则弹出。这样及时清除无效值
  3. 在遍历过程中逐步将结果记录

["239. Sliding Window Maximum" Golang]()

### "240. Search a 2D Matrix II"

- 解法: "搜索空间缩减法 Search Space Reduction"
  利用双向有序的特性，操纵一个指针的两个位置，其中一个位置的起始位置偏大，另一个起始位置偏小，比如左下角(横向偏小，纵向偏大)。然后向上、向右移动指针，直到找到答案或超出边界。整个过程都保证了未来可探索到结果，但可探索空间在不断缩小。

### "300. Longest Increasing Subsequence"

- 考点：
  - DP、LIS(最长上升子序列)
- 思路：
  - 原数组不能排序，否则无法取得最终结果
  - 设`dp[i]`表示第 i 个位置的元素与前面形成的最长子序列长度，那么`dp[i]`可以由前面遍历推算出
- 解法：
  - 有状态转移方程`dp[i]=max(dp[j])+1`，`0≤j<i && num[j]<num[i]`
  - `dp[i]`初始值设为 0，最后结果要`+1`，表示自身也算 1 的长度
- 时间复杂度：O(n^2)

["300. Longest Increasing Subsequence" DP Golang]()

["300. Longest Increasing Subsequence" DP C++]()

### "312. Burst Balloons"

- 考点：范围 DP
- 思路：
  - 把戳破气球反向理解为从两边为 1 的数组中增加气球，`增加新气球时分数 = 新增气球*左边区域最高分*右边区域分数`
  - 每个尝试新增气球被认为是一段区间内第一次插入的气球
  - 先从范围为 1 的区间开始计算，逐步扩大范围，相当于除第一次插入外，其他都已经计算出结果
- 步骤：
  - 在数组左右两遍增加一个值为 1 的元素，以便 dp 使用
  - 初始化 `dp[i][j]`表示 `[i,j]` 可以取得的最大分数
  - 第一层循环 l,1->n 表示每次处理的判断范围
  - 第二层循环 i,1->n 遍历所有位置，j = i+l-1
  - 第三层循环 k,i->j 遍历范围内所有位置，作为分割点
  - `dp[i][j] = max(dp[i][j], dp[i][k-1] + vals[i-1]*vals[k]*vals[j+1] + dp[k+1][j])`s

["312. Burst Balloons" Golang]()

### "322. Coin Change"

- 考点：
  - DP、完全背包

### "337. House Robber III"

- 解法：tree+recur+dp
  每次递归负责返回本层被选择和不选择两个结果

["337. House Robber III" Golang]()

### "338. Counting Bits"

- 解法 1: bit

  - 利用`onesCount`算法，计算 0~n 每一个数的结果
  - 时间复杂度：`O(n*32)`，假设 32 位整型

- 解法 2: bit + dp
  - 已知用`n&(n-1)`算法可以求得 n 去掉最后一个 1 的值
  - 有基本规律`0 <= n&(n-1) < n`，所以任意 n 一定能用 n-1 推导出来，这正好符合动态规划特征
  - 所以有动态转移公式`bits[x]=bits[x&(x−1)]+1`
  - 时间复杂度：`O(n)`

["338. Counting Bits" DP Golang]()

### "354. Russian Doll Envelopes"

- 解法 1: 二维元素一维化处理 + dp
  - 思路：
    - 这道题和`300. Longest Increasing Subsequence`看起来类似，就是判断每个元素有多少元素能装进去形成单调序列
    - 但是这道题是二维的，可以先对其中一维(如宽度)进行排序，然后剩下一维可以参照**LIS**算法实现
    - 注意在第一维有序基础上，第二维(如高度)要逆序，即(宽度相同的情况下)后面的保证无法装下前面的，这样避免第二维间的**后效性**

["354. Russian Doll Envelopes" DP Golang]()

### "377. Combination Sum IV"

- 方法 1：

  - dfs+memory

- 方法 2:
  - DP
  - 考点：
    - 统计排列组合，第一层状态循环、第二层物品循环

["377. Combination Sum IV" DFS C++]()
["377. Combination Sum IV" DP Golang]()

### "395. Longest Substring with At Least K Repeating Characters"

- 考点：滑窗、常数内尝试
- 思路：
  - 从"子数组连续"这个特性，想到用 SlidingWindow
  - 控制窗内元素种类是个难点。由于字母集合总数有限，干脆尝试 1 ～ 26，共 26 次，不影响时间复杂度
  - 在滑窗过程中记录几个关键参数：每种字母个数、已包含字母种类、当前未达到 k 个的字母种类数
  - 在新增、删除元素时，维护好"当前未达到 k 个字母种类数"，然后当条件符合时更新最大长度结果

["395. Longest Substring with At Least K Repeating Characters" SlidingWindow Golang]()

### "406. Queue Reconstruction by Height"

- 插入法思路：
  1. 个小的插入个大的前面，不影响排序规则，所以个头大的先插入。
  2. 同样个头的，对前面人数要求越少的应该越早进行插入操作。
  3. 输入是保证有效的，所以直接在对应位置插入即可。

["406. Queue Reconstruction by Height" C++]()

### "416. Partition Equal Subset Sum"

- 思路：dp
  - 这是一个变形的零一背包问题
  - 要想分割成两个子集，需要满足两个条件：
    1. 数组和为偶数，这样才可能平分成两个整数
    2. 任意一个元素不能超过 1 半，否则无法平分
  - 通过上述演化，可以演化为一个简易的零一背包问题(只需要保存 bool 型，无需计算价值)

["416. Partition Equal Subset Sum" Golang]()

### "448. Find All Numbers Disappeared in an Array"

- 考点：
  - array 操作
  - 约束条件：无额外空间占用
  - inplace 思想
- 思路：
  - 题目保证数字只会出现在给定范围并保证正整数，所以可以利用高位做特殊标记，比如利用负数做是否存在的标记
  - 第一遍遍历，把出现过的数字对应的下标元素置为负数，那么剩下未标记为负数的位置对应的值就未出现过
  - 对应的，访问每个元素时，要注意把负数恢复为正数，然后再取对应下标
  - 第二遍遍历，将仍然为正数的下标对应的数值记录，作为结果返回

["448. Find All Numbers Disappeared in an Array" Golang]()

### "456. 132 Pattern"

- 思路：
  - 只要直到任意位置左边最小值和右边最接近的较小值，就可以判断是否有符合条件的值
  - 先从左向右遍历，记录每个位置左边的最小值
  - 再从右向左遍历，利用单调栈记录比当前值更大的值，当较小值出栈时进行一次条件判断

### "461. Hamming Distance"

- **汉明距离**广泛用于多个领域，在编码理论中用于错误检测，在信息论中量化字符串之间的差异。两个整数之间的汉明距离是对应位置上数字不同的位数。
- 一般语言都会自带函数，这里要自己实现
- 解法：
  1. 两个数取异或，则结果中的 1 就是两者差异点，只要统计这些差异点数量即可
  2. 比遍历更快的方式是利用`去除x最右边的1 = x & (x-1)`，然后只需几次就能统计完毕

["461. Hamming Distance" Golang]()

### "474. Ones and Zeroes"

- 考点：
  - 零一背包+二维逆序推演
- 技巧：
  - 本题只求最大值，所以不用精确推算结果，在保证无**后效性**的同时，始终保持数组靠后为最大值即可

### "494. Target Sum"

- 考点：
  - 概率 DP、偏移重整
- 思路：
  - 尝试 1、2、3 个元素的组合，可发现状态转移公式
  - 利用`sum <= 1000`的特性，用`+1000`的方式规避负数下标出现

### "518. Coin Change 2"

- 考点：
  - 概率 DP、完全背包版
- 思路：
  - 标准完全背包算法

["518. Coin Change 2" Golang]()

### "560. Subarray Sum Equals K"

- 思路：
  - **子数组和**结合**累加数组**思想，可以把问题化简为`数组中 A - B = k <=> A - k = B`
  - 由于可以重复统计，并且统计结果数是递增的，可以利用 map 进行累加，`map[A-K]==count`
  - 起始处，要有`map[0]=1`，以便统计独立元素直接匹配的情况

["560. Subarray Sum Equals K" Golang]()

### "664. Strange Printer"

- 解法 1: 范围 DP + DFS + mem
- 思路：
  - dp[i][j] 表示 i~j 范围最小打印的次数
  - 对所有范围递归求解，每个范围尝试每个点为分割点
  - 如果分割点和末尾字符一致，则末尾字符可以少打印一次，`dp[i][j] = min(dp[i][j], dfs(s, i, k) + dfs(s, k+1, j-1))`
  - 在递归开始要先对当前 dp 赋一个保底值

["664. Strange Printer" DP]()

### "688. Knight Probability in Chessboard"

- 解法：dp
  与之前的"935. Knight Dialer"原理相同，但是增加了两点复杂度：
  1. 可落子的区域变大，可能性更多，需要用两个大的棋盘来演化，并且统计数值会非常大。
  2. 答案要求计算概率而不是统计数值，由于统计数值过大，中间计算过程就会溢出(超出 double 型)，
     所以棋盘要存储每一位置的概率(double)，并且每次引用上一次演化结果时要"/8.0"(上一次位置到当前位置有 8 种可能结果)而不是累计到最后"/pow(8.0, K)"。

### "765. Couples Holding Hands"

- 考点：
  - DSU、Chart、公式推导
  - 假设座位下标从 0 记，1、2 两个位置是不能拉手的，这样将导致首尾两个位置不可能拉手，最后只能所有人移动，不可行。
    所以每两个座位被分为一组，所有组之间是独立的关系，可以将座位数组看作一个图，图的元素就是一组关联的座位
  - 尝试 1、2、3 组情侣乱坐的情况，可以分析得出`最小移动次数=乱坐情侣组数-1`，比如有 3 组乱坐需要移动 2 次。
    (乱坐的组可以形成一个有序的链式关系)
- 思路：
  - 遍历每组座位，通过`x/2`判断是否属于情侣，如果不是则认为乱坐
  - 用 DSU 把乱坐的人关联起来，最终形成多个乱坐的集合
  - 遍历每个乱坐集合的总数，并统计最终移动的最小次数
  - 进一步，可以简化最终结果为`总组数-集合数(包括乱坐和没乱坐的)`

### "818. Race Car"

- 解法 1: BFS + memory

  - 把"位置+速度"作为一个基本单元，放在 queue 里进行 BFS，每次遍历尝试继续加速或减速
  - 由于规模较大"10000"，所以要进行 memory 剪枝，以"位置+速度"作为 key 保存在 map 中
    - 由于规模在`1~10000`之内，同时速度是"2^n"，可以用一个 32 位表示一个 key"位置+速度+速度方向"，
      但是追求极致节省内存没必要，可以用一个结构体或 string 表示一个 key
  - 另外还要限制 BFS 范围：当`范围 > 2*target`时，就不再添加元素
    - 到达最终位置的路径可以是一个"蛇形"过程，超出太远就没有意义了

- 解法 2: DP

### "912. Sort an Array"

- 快排、归并、堆排序都能在 nlogn 时间复杂度内完成
- 快排思路：
  1. 将待排数组分为两部分，设定一个中值，要求遍历过程中左边小于中值、右边大于中值
  2. 设定两个指针，通过交替遍历向中心靠拢，使左右符合要求
  3. 递归左边、右边，最终达到整体有序
- pivot 取值改进思路：
  - 三数取中法(效果最好)
  - 随机取值法

["912. Sort an Array" QuickSort C++]()

["912. Sort an Array" QuickSort Golang]()

### "935. Knight Dialer"

- 解法：
  此题看似有数学规律，实际只能靠基本演化逻辑，通过动态规划推导出最后结果。
  每次演化都要从之前的某个位置过来，所以只需要两组位置计数不断演化即可。

### "974. Subarray Sums Divisible by K"

- 考点：
  - 连续整数组成的子序列，可以利用 sum(i,j)来快速表达
  - Math: 同余定理，`(a-b)%m == 0 <=> a%m == b%m == k`，如果两数之差可以被 m 整除，那么两数分别对 m 取余的值相同
  - 利用 HashTable 对已存在元素累加
- 思路：
  - 设 P[i] 是前面 i 个数累加和，`sum(i,j) == P[j]-P[i-1]`
  - 根据同余定理`sum(i,j)%K == 0 <=> P[j]%K == P[i-1]%K`

["974. Subarray Sums Divisible by K" HashTable Golang]()

### "1004. Max Consecutive Ones III"

- 思路：
  关键字`contiguous`，所以考虑用 sliding window

### "1036. Escape a Large Maze"

- 考点：bfs 稀疏矩阵 极限思想 几何
- 题干分析：
  - 坐标最大值`10^6`，说明矩阵范围巨大
  - `blocked.length <= 200`，说明有效数据只占一小部分，是一个稀疏矩阵，不能用传统矩阵方法标记
- 思路：
  - 矩阵太大，无法用二维数组标记，应该用 set 记录已访问位置
  - 即使用 set 记录，空间复杂度也太大。可以利用几何特性缩小 bfs 范围，200 个 block 挡住的最大闭合范围也就是一个三角形，其中两条边是边界，block 充当封住边界的边。这样可以估算出数量为 20006(勾股定理、根号 2)，因为 block 本身也占 200 容量，所以实际范围内单元数`<20000`，只要遍历元素超过 20000，说明 visited 范围一定会延伸到 block 外
  - "封口"可能发生在任何一方，需要同时检测出发点和目标点的 bfs 范围都`>20000`时，才能判定为 true

["1036. Escape a Large Maze" bfs Golang]()

### "1054. Distant Barcodes"

- 思路：
  - 尽量让相同元素相互隔开
  - 数量越多的元素越优先考虑，因为它们更"容易"在一起
- 解法 1：
  1. 用 Map 统计元素数量
  2. 将元素按出现频次放入最大堆
  3. 准备一个结果数组
  4. for 循环堆中元素，循环每个位置防止
  5. 先放奇数位，放满后放偶数位(下标移到 1)
- 解法 2:
  解法 1 中用到了 Heap，如果不方便使用，可以对统计结果再排序一次，然后按照从大到小处理一遍即可
  - 如果 Heap 元素只在最后 pop 一次，那 Heap 完全可以用数组替代，做一次排序即可
- 技巧点：
  - 由于保证存在答案，只需要让元素自己隔离就可以，不用考虑其他元素
  - 在奇偶转换时，输出数组的前后位置也会自动隔离元素，所以不用担心出问题

### "1128. Number of Equivalent Domino Pairs"

- 思路：
  - 只需要对"相同"的元素累加统计即可
  - 由于每个分量值`<9`，可以用一个排过序的两位数表示
  - 可以利用一个 100 个元素的 bucket 统计，这样就无需排序一次遍历就统计出来

### "1178. Number of Valid Words for Each Puzzle"

- 考点：
  - 本题重点是元素数量巨大：`1 <= words.length <= 10^5`、`1 <= puzzles.length <= 10^4`，
    所以无法对所有 word 和 puzzle 两两比较，需要多处优化
  - 分治，分两个阶段操作得到结果
  - 二进制压缩保存特征值
  - 极端数量下根据特征合并同类项，降低整体时间复杂度
  - bitcount 算法
  - bitSubset 算法
- 思路：
  - 首先对 word 遍历，提取特征值为 bitmap，如果 bitcount 符合要求则累加到 map 中
  - 由于数量太大，而 puzzle 元素数较小(<=7)，所以直接对 puzzle 的二进制子集进行遍历

["1178. Number of Valid Words for Each Puzzle" BinaryOperation Golang]()

### "1438. Longest Continuous Subarray With Absolute Diff Less Than or Equal to Limit"

- 解法 1:
  利用 C++ STL 的 multiset 实现。在遍历过程中维护最大、最小值及中间元素。如果是其他语言，如 Golang 需要手写 Treap 结构。

  - 时间复杂度 O(nlogn)，每次维护 tree 需要 logn，空间复杂度 O(n)

- 解法 2: 滑窗+单调队列
  - 思路：
    - `continuous`、`subarray`表示用连续的元素统计结果，考虑用 SlidingWindow。
    - 在窗口移动过程中要记录 max、min 两个值，用来判断当前元素是否超范围
    - 在窗口 left 端移动时，要调整 max、min 值
    - 上面保持 max、min 值，同时要保持元素的先后顺序关系，考虑用两个 MonotoneQue 来实现
  - 流程：
    1. for 循环所有下标，作为 right 值，保持 minQue 和 maxQue 最新状态
    2. 如果当前范围不匹配 limit 则将窗口移动，同时判断是否有元素出队列
    3. 记录 right - left 最大值，直到循环结束

["1438. Longest Continuous Subarray With Absolute Diff Less Than or Equal to Limit" SlidingWindow+MonotoneQueue Golang]()

### "1463. Cherry Pickup II"

- 考点：
  - DP
- 思路：
  - 用三维数组 dp[x][y1][y2]表示某行下机器人 1、机器人 2 的位置下已获取的最大分数
  - 转移方程：`dp[x][y1][y2] = max(dp[x][y1][y2], dp[x-1][y1-1~y1+1][y2-1~y2+1])`
  - 注意设置初始行，然后开始迭代。考虑到初始行可能是 0 值，所以 dp 初始值设置为无效值 0
  - 最后需遍历找到最后一行中的最大值返回，注意需要对 y1、y2 两个变量进行遍历
  - 优化：可以只用两行数据进行迭代