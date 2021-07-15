---
layout: post
title: "LeetCode Question Category"
date: 2020-12-27 08:00:00 +0800
tags: Algorithm Leetcode
---

![LeetCode](/assets/images/2020-12-27-LeetCode_Category_1.jpg)
记录 LeetCode 题型分类及解题思路

### "7. Reverse Integer"

- 解法：math
  - 基本方法是通过`%`、`/10`循环取得结果
  - 难点是如果超出 Int32 范围需返回 0，并且要求不允许使用 int64 类型，这样无法对最终结果做范围判断
  - 可以利用`ret*10`处理前对`math.MinInt32/10`和`math.MaxInt32/10`做判断，预判最终结果

["7. Reverse Integer" math]()

### "11. Container With Most Water"

- 解法 1(推荐)：

  - 高度、距离都是影响水量的值，其中高度的权重更大，所以在循环缩小距离时保持较高的高度让高度较小的一方遍历。

- 解法 2：
  - 利用动态规划法，分别计算当前位置左边最大高度、右边最大高度，算法较易理解，但是需要占用额外空间。

### "12. Integer to Roman"

- 解法：DC
  - 思路：
    - 根据当前 10 进制尾数选择字符
    - 根据不同情况拼接字符

["12. Integer to Roman" DivideAndConquer Golang]()

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

### "35. Search Insert Position"

### "49. Group Anagrams"

- 思路：
  利用 Anagram(异位词)特性，统计每个词的字母出现次数，生成一个"指纹"，然后通过 HashMap 收集相同指纹的词

["49. Group Anagrams" HashTable]()

### "33. Search in Rotated Sorted Array"

- 思路：
  这道题是在二分查找的基础上，增加一层排除法逻辑，使得每次都能缩小一半查找范围。
- 步骤：
  - 最外层 for 循环，条件是`l<r`
  - for 内第一层 if 如果左右其中一部分是有序的，则认为另一部分无序，根据有序的继续判断
  - 第二层 if 如果 targe 在范围内，则继续缩小范围，否则丢弃当前范围
- 要点：
  - 二分查找时，要根据判断条件的`=`情况，缩小范围。如果匹配`=`则原有边界要保持，如`if tar<=nums[mid] {r = mid}`

["33. Search in Rotated Sorted Array" Binary Search]()

### "34. Find First and Last Position of Element in Sorted Array"

- 思路：lowerbound
  这道题是在二分查找的基础上，进一步利用算法细节特性："二分查找的结果是重复元素第一次出现的下标"。
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

### "61. Rotate List"

- 难点：
  - 题目最后列出限制条件`0 <= k <= 2 * 10^9`，所以要先遍历一遍计算总结点数 n，然后用用总节点数取模计算出跳转数

### "64. Minimum Path Sum"

- 思路：这道题有较好的 DP 解决方案，所以用 DFS 时性能相比较差就会超时。解决思路：从右下角开始动态规划，计算每个下一个位置可能的最小值。利用原空间就可以完整 DP 过程。

### "65. Valid Number"

- 解法：FSM 有限状态自动机
  - 思路：
    - 本题不需要计算结果，只需要求出最后是否有效，所以简化了思路
    - 可以采用正则表达式，也可以利用二级 map 实现有限状态自动机

["65. Valid Number" StructDesign]()

### "69. Sqrt(x)"

- 二分查找法:

  - 在 1~x 之间进行二分查找，条件为`m > x/m`，注意这里没有用`m * m > x`因为可能溢出，
  - 使用`/`会比乘法慢一些，但是二分查找的时间复杂度是 O(logn)所以问题不大。
  - 使用二分查找法效率比牛顿迭代法稍高。

![Newton Iterate Algorithm](/assets/images/2020-12-27-LeetCode_Question_Category_69.jpeg)

- 牛顿迭代法：
  - 设函数 f(x)=x^2- t，初始随意取一点 x0 = t，则与函数有一个焦点，对这点求导，得到切线与 x 轴得新的 x1 = (x0/2)+t/(2\*x0)。
  - 不断重复上述步骤，当 f(xn) = xn^2-t = 0 时，xn 就是答案。

["69. Sqrt(x)" Math C++]()

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

### "74. Search a 2D Matrix"

- 解法 1: 逐步贴近法
  - 利用横纵分别有序的特性，以某个角(如左下)开始遍历，先向上遍历找到刚好小于等于目标值的位置，
    然后向右遍历找到目标
- 时间复杂度: O(m+n)

- 解法 2: 两次二分查找
  - 先纵向二分查找，先找到刚好小于等于目标值的位置
  - 再横向二分查找，找到目标
- 时间复杂度：O(logm+logn)

- 解法 3: 拼接二维数组为一维，一次二分查找
  - 由于每行一定上一行，可以将每行首位拼接，组成一个有序数组
  - 利用一个一维坐标进行二分查找
- 时间复杂度：O(log(m+n))

["74. Search a 2D Matrix" Array Golang]()

### "78. Subsets"

- 迭代法：
  可以将迭代过程看做：`[[]] -> [[],[1]] -> [[],[1],[2],[1,2]] -> [[],[1],[2],[1,2],[3],[1,3],[2,3],[1,2,3]]`
  每次将前面的内容复制到结尾，并将新的数字加入进去。

["78. Subsets" C++]()

- 回朔法：
  设置一个集合切片，每一个元素可以选取或不选取

["78. Subsets" Golang]()

### "81. Search in Rotated Sorted Array II"

- 解法：二分查找
  - 思路：
    这道题和`33. Search in Rotated Sorted Array`思路一致，只是会有重复元素导致无法判断哪一边。
    可以在`nums[l]==nums[mid]==nums[r]`的时候，两边同时缩小范围`l++;r--`解决

### "82. Remove Duplicates from Sorted List II"

- 思路：
  - 先创建一个 dummyHead，Val 值为`-101`
  - preTail 指向前面的最后一个元素
  - 一旦判断当前元素存在重复，则 for 循环跳过所有相同值元素

["82. Remove Duplicates from Sorted List II" Golang]()

### "84. Largest Rectangle in Histogram"

- 这道题的**单调栈**解题思路（时间复杂度 O(n)）很重要，在很多其他题中都会用到。
- 基本原理：
  1. 从左向右遍历，并不断将高度值的下标 push 入 stack。
  2. 当某一位置 height 值缩小时，那么之后就很难计算前面的面积了，所以就要把前面比当前值高的 stack 出栈，并计算前面的面积。
  3. 注意，计算面积时，宽度要当前下标和前面的"背影"相减，所以栈底是-1。

["84. Largest Rectangle in Histogram" Golang]()

### "88. Merge Sorted Array"

- 思路：归并排序
  - 利用原有数组容量和顺序特性，从大到小进行归并排序
  - 时间复杂度 O(n)，空间复杂度 O(1)

### "90. Subsets II"

- 解法 1: backtracking + memory

  - 思路：
    - 利用 backtraking 对每个元素执行"取/不取"操作
    - 将所有元素拼接成字符串作为唯一标识，用`map[string]struct{}`进行重复判别
    - 由于乱序下元素顺序不同可能影响集合一致性判别，所以要先进行一次排序
  - 缺点：
    由于大量字符串拼接操作，时间、空间复杂度都存在较大浪费

- 解法 2: bit mask
  - 思路：
    - 由于是任意子集，所以原始元素顺序不重要，先对元素做排序以便后续操作
    - 对于长度为 n 的 nums 中所有元素可以"选"和"不选"，可以看作是 n 位二进制数的每一位进行选取或不选操作。
      可以设一个 n 位的 mask 二进制数，则 0/1 序列对应的二进制数量正好为`[0,(2^n)-1]`
      (题目的 n 较小，也使得 mask 方案成为可能)
    - 考虑`[1,2,2]`这种情况，取下标"0 1"和"1 2"的结果是一样的，都是`[1,2]`。
      可以推导出如果元素 x 与前面的元素 y 相同，如果 y 没有取而取了 x，则包含 x 的子集必然会出现在包含 y 的所有子集中，
      所以可以跳过这种情况，继续下一个 mask 处理
  - 优点：
    利用二进制操作性能较高

["90. Subsets II" Golang bit mask]()

- 解法 3: backtracking
  - 思路：
    - 以 backtracking 为基本思路，结合上面`[1,2,2]`模式的识别跳过不必要的处理
    - 注意 Go 中 append 时为了避免各递归函数相互影响，要构造新的 slice

### "91. Decode Ways"

- 解法 1: DFS + Memory
  - 思路：
    - 每个位置可以取自己，也可以取自己和后面一个字符，用 DFS 即可
    - 每次如果可以取，那一定符合`0< x <= 26`的条件，把条件字符串放入 set 可以快速判断
    - 递归函数返回其对应的结果数，由调用者负责将两种情况相加
    - 如果没有 Memory 会超时，所以需要加上

["91. Decode Ways" DFS Golang]()

- 解法 2: DP
  - 思路：
    - 从前到后逐步解码，只有前面解码成功后面的解码才成功
    - 设`dp[i]`表示第 i 个元素，有两种情况：
      如果当前值有效`s[i] != '0'`，则继续累加前面数字；
      如果当前前面有效且和前面可拼接为 26 以内的数，则继续累加更前面数字；
  - 要点：
    - `dp[0]==1`，空字符串可以编码为空，也是一种编码结果
    - 由于`dp[0]`要占一个位置，所以 i 要顺延一位，对应 i(从 1 遍历) 的字符为`s[i-1]`
    - 实现后发现只需要关注三个 dp 值即可，所以可以优化为 O(1)空间复杂度

["91. Decode Ways" DP Golang]()

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

### "136. Single Number"

- 解法：按位状态机
  - 题目要求`O(1)`空间复杂度，所以需要利用异或运算按位统计出结果

["136. Single Number" BinaryOperation Go]()

### "137. Single Number II"

- 解法：按位状态机
  - 基于"136"的解法，利用数字电路(与、或、非、异或)推导，利用两个变量保存运算过程值

["137. Single Number II" BinaryOperation Go]()

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

### "153. Find Minimum in Rotated Sorted Array"

- 解法：BinarySearch
  - 思路：
    - `Rotated Sorted Array`的特点是，如果取 mid 位置，则一边是升序而另一边是降序，假设为`[2,3,1]`
    - 通过不断排除`升序子区域的右边`和`降序子区域的左边`就可以找到最小值
  - 要点：
    - 由于存在"最大值突变最小值"的转换点，所以丢弃左边区域时要`l = mid+1`，这样最终 l 就是结果了

### "154. Find Minimum in Rotated Sorted Array II"

- 解法：BinarySearch
  - 思路：
    类似"81. Search in Rotated Sorted Array II"，在无法判断哪边有序时，左右范围都缩小一下

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
  这种问题先简化思路，假设有两个链表，分别为:`A->C->D`和`B->C->D`，则 C 是交汇点。
  如果按照上面方法使跳转路径相交叠，则两个指针交汇前走的距离分别是：`AC+CD+BC`和`BC+CD+AB`，可以看到，这两个距离是完全相等的
- 这个解法的优点是空间复杂度为 O(1)

["160. Intersection of Two Linked Lists" LinkedList]()

### "169. Majority Element"

- 思路：
  这类求**Majority(众数)**的问题，可以用**Boyer-Moore(摩尔投票法)**解决，基本思路就是"小弟够多不怕对砍"。
- 步骤：
  1. 对某个数进行计数并记录该数，如果遇到这个数则计数加一;
  2. 如果不是这个数则计数减一，如果为零则重新计数。
  3. 最后如果计数大于 1，则遍历数组对找到的数计数，如果总数超过了 1/2，则找到了目标。

["169. Majority Element" Array]()

### "179. Largest Number"

- 思路：
  - 可以对 nums 中的元素进行排序，排序方法两两前后组合一下，判断顺序，此顺序具有传递性，所以只需要快排就可以
  - 一般可以利用语言中的快排算法直接实现，在 Go 中可以利用`sort.Slice(interface{}, func (a, b int) bool)`;
    字符串转换同样利用现有函数，在 Go 中可以用`strconv.Itoa(int)`
  - 比较的关键是设计两个系数，比如把`a`置于前面时对应的数值是`factorA * a + b`，factorA 起始值为 10 并循环增大到超过`b`；
    b 的系数取法类比 a
    由于题目条件`0 <= nums[i] <= 10^9`，所以数值可能超过 int32，因此要利用 int64 为比较值
  - 排序完毕要检查`nums[0]`，如果`0`说明整个数字以 0 开头，则结果无需字符串拼接，直接返回"0"
  - 最后返回结果要拼接字符串，注意避免字符串连续`+`导致内存频繁申请；在 Go 中可以利用`string([]byte{})`

["179. Largest Number" Array]()

### "198. House Robber"

- 解法：DP
  - 思路：
    - 推演前 4 个房间的过程就可以推出 DP 转移方程
    - 设 dp[i] 为到第 i 个房间为止可抢的最大数额
    - dp[0] 第一间房，数额只能是 nums[0]
    - dp[1] 第二间房，有两种情况：如果抢前面不抢当前，是`dp[0] <=> nums[0]`；
      如果抢当前不抢前面，是`dp[-1]+nums[1]`这里`dp[-1]`表示更往前的结果是不存在的所以`==0`。
      两种情况综合`dp[1] = max(nums[0], nums[1])`
    - dp[2] 第三间房，同样两种情况：如果抢前面不抢当前，是`max(dp[0], dp[1])`;
      如果抢当前不抢前面，是`dp[0]+nums[2]`;
      可推导出转移方程：`dp[i] = max(max(dp[i-1],dp[i-2]), dp[i-2]+nums[i])`
    - 由于只需要历史的两步，所以可以进一步缩减为固定变量演进

["198. House Robber" DP Golang]()

### "208. Implement Trie (Prefix Tree)"

- 解法：基本的 TrieTree 实现

["208. Implement Trie (Prefix Tree)" TrieTree Golang]()

### "212. Word Search II"

- 解法：TrieTree + Backtracking
  - 思路：
    - 因为单词数量很大`3*10^4`，所以如果都用 DFS 去尝试肯定会超时
    - 可以利用 TrieTree，先把所有单词记录为 TrieTree 字典
    - 然后再利用 Backtracking 在 board 中进行检索
  - 技巧：
    - 整个过程的关键是 TrieTree 的结点，所以无需封装，直接暴露操作就可以
    - 由于 TrieTree 的结点一般只标记是否找到，不便与本题，所以可以增加一个`word string`记录完整字符串，以便存入结果

### "213. House Robber II"

- 解法：DP + 环状 array 技巧
  - 思路：
    - 基本 DP 思想是基于"198. House Robber"，一定要先搞懂
    - 由于本题环状结构的特殊性，处理边界条件非常麻烦...
    - 干脆将数组分为两部分："不包含第一个房间"、"不包含最后一个房间"，然后分别计算再取结果的最大值即可
  - 改进：
    - 储存时只保存下标节省更多容量
    - 在删除 heap 元素时需要 O(n)的遍历，效率较低，
      可以在 heap 中保存 end 时间，以便 O(logn)删除 heap 元素
      (每次先将当前 end 之前失效的元素删除)

["213. House Robber II" DP Golang]()

### "218. The Skyline Problem"

- 解法：LineSweep
  - 思路：
    - 将一栋楼的坐标点拆解为 start 和 end，然后顺序遍历处理，遍历过程中维护一个 Heap，用于计算当前轮廓点
    - 遍历过程中维护一个 maxHeight，如果当前是 start，则入堆；如果当前是 end，则删除一个同高度；
    - 在遍历过程中，记录会造成轮廓变化的点
  - 要点：
    - 为了便于 start 和 end 排序，将 start 按负数处理，这样 start 会排到前面
    - 对于同一坐标下最高的 start 会加入到最终结果数组，所以同一坐标下高度越高的 start 点应排在前面，按负数处理正好符合这个情况
    - 遍历处理 end 点时，要从 Heap 中找到一个相同高度元素删除，这时要用到 heap.Remove 接口

["218. The Skyline Problem" LineSweep Golang]()

### "220. Contains Duplicate III"

- 解法 1: SlidingWindow + OrderedMap
  - 这种解法难度对于 C++属于"normal"，可以直接利用`set<int>`的高性能红黑树；对于 Golang 属于"hard"，需要手动实现 Treap 或 Skiplist
  - 思路：
    - 以 SlidingWindows 保持对下标范围的控制，始终保持窗口内元素下标合法
    - 以 OrderedMap 作为 value 的存储，可以快速判断顺序前后两个元素是否符合条件
  - 时间复杂度：O(nlog(min(n,k)))；空间复杂度：O(min(n,k))

["213. House Robber II" OrderedMap RedBlackTree C++]()
["213. House Robber II" OrderedMap Treap Golang]()

- 解法 2(推荐): SlidingWindow + Bucket + HashMap
  - 由于比较综合，这个解法属于"hard"
  - 思路：
    - 已知 k 为有效下标范围、t 为有效值范围
    - 设 x 为当前遍历元素、y 为下标有效范围内另一元素，如果 y 在范围`[x-t, x+t]`则认为命中
    - 我们设每个桶容量为`t+1`这样保证桶内元素一定命中
    - 如果没有直接命中，也要检查一下旁边的两个桶，判断是否可能命中，这样范围仍然是常数的
  - 要点：
    - 由于本题范围较大，需要用 Hash 存储桶，所以设定一个函数`getID(x, w int) int`，
      传入`x`值和桶容量`w`，通过`x/w`计算得出桶的下标。
    - `getID`函数的负数处理比较特殊，为了避免`+0和-0`被重复计算，需要在负值时返回`(x+1)/w - 1`

["213. House Robber II" Bucket Golang]()

### "229. Majority Element II"

- 思路：
  **众数问题**用**摩尔投票法**
- 不同点：
  1. 统计的数成了两个
  2. 最后统计时，最终结果元素要超过统计结果的 1/3
- 要点：
  1. 遍历统计时，只要其中一个元素可以累加/初始化(count 0->1)，则 continue
  2. 如果无法累加/初始化，则两个元素计数都要减

["229. Majority Element II" Array Golang]()

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

### "250. Count Univalue Subtrees"

- 解法，自创"探针法"
  - 思路：
    递归调用、后根遍历，每个节点向孩子传递一个整型引用(探针)，孩子值与父值不同或者孩子值与子孙不同，探针引用都要增 1，
    注意左右孩子分别建立探针，不要相互影响。

### "264. Ugly Number II"

- 解法：Math+DP
  - 思路：
    - 可以把未来所有可能数字放在堆，然后逐步取出，但是这种方法太浪费空间
    - 对于任意有效丑数 x，$x=(k_{x}*2)*(k_{y}*3)*(k_{z}*5)$，其中$k_{x}$表示正数系数
    - 设 dp[i]为第 i 个丑数，每次都要尽量找到最接近的下一个有效值，有`i > kx && i > ky && i > kz`，
      每次演化的值可以从 dp 历史值取得$dp[i]=min(dp[k_{x}]×2,dp[k_{y}]×3,dp[k_{z}]×5)$
    - 所以只需要逐步提高最小的系数，就可以推演出最终结果

["264. Ugly Number II" Math Golang]()

### "274. H-Index"

- 解法 1：Sort + BruteForce
  - 思路：
    - 被引用的次数越多的文章，越容易实现要求
    - 先按引用次数排序，然后倒序遍历，遇到不符合的就退出循环
    - 由于引用次数这个数值本身较大容易满足，所以把文件个数作为最后返回的结果
  - 错误思路：
    - 由于排序后的数组对于`H-Index`满足具有二分查找特性，所以可能会想到用二分查找。
      但是排序本身时间复杂度是`O(nlogn)`，所以总的时间复杂度是`O(nlogn + logn) == O(nlogn)`，
      总效率没有优化，而且增加了算法复杂度

["274. H-Index" BruteForce]()

- 解法 2: CountSort + BruteForce
  - 思路：
    - 由于引用次数往往较大，而符合条件的文件数较小，结果为两者取其小
    - 先确定最大数量为输入数组长度，利用 CountSort 统计某引用数量出现次数
    - 遍历统计已满足的数量，找到最大数量
  - 时间复杂度`O(n)`，空间复杂度`O(n)`

["274. H-Index" Sort]()

### "275. H-Index II"

- 解法： LowerBound
  - 思路：
    - 基于"274. H-Index"的计数逻辑，已经有序后可以利用二分查找算法实现
    - 而本题目刚好符合 LowerBound 算法

["275. H-Index II" BinarySearch]()

### "279. Perfect Squares"

- 解法 1：DP

  - 思路：
    - 可以看称"完全背包"问题，目标数字`n`就是背包容量，`x^2`可以看作是物品，物品数量数量不限
    - 附加条件：用**最少**的物品个数，**刚好**填满背包
    - 可以先把备选数字筛出来，放入数组作为物品，数字平方的最大值不超过`n`
    - 第一层循环物品，第二层循环状态，记录可到达目标的最小个数

- 解法 2: DP
  - 思路：
    - 某个数字 n 对应的最小值可以由前面某个位置推算过来，所以可以对 n 做个 DP 演化
    - 演化时，无需把物品提前计算出来放到数组，可以直接利用`x^2;x++`遍历"物品"

["279. Perfect Squares" DynamicPlanning]()

### "300. Longest Increasing Subsequence"

**LIS**是动态规划的常见问题

- 考点：

  - 一维数组二维 DP、LIS(最长上升子序列)

- 解法 1：
  - 思路：
    - 原数组不能排序，否则无法取得最终结果
    - 设`dp[i]`表示第 i 个位置的元素与前面形成的最长子序列长度，那么`dp[i]`可以由前面遍历推算出
    - 有状态转移方程`dp[i]=max(dp[j])+1`，`0≤j<i && num[j]<num[i]`
    - `dp[i]`初始值设为 0，最后结果要`+1`，表示自身也算 1 的长度
  - 时间复杂度：O(n^2)

["300. Longest Increasing Subsequence" DP Golang]()

- 解法 2: DP + Lowerbound
  - 思路：
    - 贪心思维：在已有最长子序列基础上用尽可能小的新元素替换旧元素，使得未来上升时长度可以更长
    - 维护一个数组`d[i]`表示长度为 i 的最长上升子序列末尾元素的最小值，
      len 记录最长上升元素数、起始 len 为 1、`d[1]==nums[0]`
  - 步骤：
    - 遍历 nums，更新 d，如果`nums[i] > d[len]`，则`d[len+1]==nums[i]`;
      否则在`1~len`中找到`d[i-1] < nums[j] < d[i]`并更新`d[i]=nums[j]`
    - 由于 d 本身的有序性，可以利用`Lowerbound`算法查找
  - 时间复杂度: O(nlogn)

["300. Longest Increasing Subsequence" BinarySearch Golang]()
["300. Longest Increasing Subsequence" BinarySearch C++]()

### "307. Range Sum Query - Mutable"

- 解法：BIT
  - 思路：
    - 标准 BIT 的应用场景，频繁对数组更新、求和
  - 注意：
    - 每次 update 时不是 delta 而是 value，需要求一下 diff 值

["307. Range Sum Query - Mutable" BinaryIndexedTree C++]()

### "312. Burst Balloons"

- 解法(推荐)： DFS + Memory

  - 思路：
    - 用 memo[i][j]记录[i,j]区间内的最大值
    - 假设打破气球的位置是 k，`i <= k <= j`，我们假设 k 是区间内最后一个打破的气球，这样容易计算
    - 然后套用区间 DP 模板(DFS+Memory)实现
  - 注意：
    - 区间内位置 k 最后一个打破的分数，要用到区间外的元素`i-1`和`j+1`，所以需要区间边缘虚拟元素处理
    - 区间内的分值由三部分相加得出：`[i,k-1] + k最后一个打破分数 + [k+1,j]`

["312. Burst Balloons" DFS+Memory Golang]()

- 解法：区间 DP
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

["312. Burst Balloons" DP Golang]()

### "322. Coin Change"

- 考点：
  - DP、完全背包

### "324. Wiggle Sort II"

- 解法：
  1. 利用**nth_element**函数在 O(n)时间内，在中间位置左边排序，保证左边比右边小，并返回中间位置的值。
  2. 将左边有序的较小数的奇数下标位置替换为右边特定位置的较大数(注意循环结束条件是`j <= k`，判断条件是 j 与 mid 的关系)。
  3. 使用`(1+2*(i)) % (n|1)`对所有奇数位置比较和替换，后面的`(n|1)`来保证不论 n 是奇偶，取模后都是奇数位置。
  4. 使用宏函数，避免过多重复代码。

["324. Wiggle Sort II" Sort C++]()

### "332. Reconstruct Itinerary"

- 解法：欧拉路径 Hierholzer Algorithm
  - 思路：
    - 当前问题正好是每张票用一次并全部用掉，符合欧拉路径

["332. Reconstruct Itinerary" Graph C++]()

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

["338. Counting Bits" DynamicPlanning Golang]()

### "354. Russian Doll Envelopes"

- 解法 1: 二维元素一维化处理 + dp
  - 思路：
    - 这道题和`300. Longest Increasing Subsequence`看起来类似，就是判断每个元素有多少元素能装进去形成单调序列
    - 但是这道题是二维的，可以先对其中一维(如宽度)进行排序，然后剩下一维可以参照**LIS**算法实现
    - 注意在第一维有序基础上，第二维(如高度)要逆序，即(宽度相同的情况下)后面的保证无法装下前面的，这样避免第二维间的**后效性**

["354. Russian Doll Envelopes" DP Golang]()

### "363. Max Sum of Rectangle No Larger Than K"

- 解法 1: 二维压缩 + 暴力计算
  - 思路：
    - 先利用"前缀和"对每一行压缩
    - 利用相同方法，再对每一列压缩
    - 之后可以利用二维两个坐标点确定一个矩形范围，遍历所有矩形计算出结果
  - 要点：
    - 注意最后计算范围结果时，要将重复减去的范围值加回来

["363. Max Sum of Rectangle No Larger Than K" Array]()

- 解法 2:

### "368. Largest Divisible Subset"

- 解法：DP + LIS + 反推 DP 结果
  - 思路：
    - 本题的基本思路参照`300. Longest Increasing Subsequence`，必须先掌握 LIS 算法
    - 由于题目对结果顺序没有要求，所以先对原数组排序，以便进行 DP 处理
    - `dp[i]`表示到`nums[i]`为止的最大长度，利用整除的数学特性逐步迭代
    - 在迭代过程中要记录最大长度和对应的最大值，以便最后反推，计算出最终结果

["368. Largest Divisible Subset" DP Golang]()

### "374. Guess Number Higher or Lower"

- 解法：BinarySearch
  - 要点：
    - 最大值范围`(2^31)-1`，可以变形模板`r = n`，最后返回`r`

["374. Guess Number Higher or Lower" BinarySearch]()

### "377. Combination Sum IV"

- 方法 1：

  - dfs+memory

- 方法 2:
  - DP
  - 考点：
    - 统计排列组合，第一层状态循环、第二层物品循环

["377. Combination Sum IV" DFS C++]()
["377. Combination Sum IV" DP Golang]()

### "380. Insert Delete GetRandom O(1)"

- 解法：HashTable + Array 实现 O(1) 数据结构
  - 思路：
    - 元素放在 Array 中，以便方便的随机选取
    - 利用 HashMap 实现 O(1)查询，key 保存元素值，value 保存元素在 Array 中的下标

["380. Insert Delete GetRandom O(1)" StructDesign]()

- 解法：二维 HashMap + Array 实现可重复 O(1) 数据结构
  - 要点：
    - remove 函数的实现较为复杂，基本思路是：记录最后一个元素值、删除当前元素、删除最后一个元素旧下标、新增最后一个元素新下标、删除多余内容
    - 如果删除的当前元素正好是最后一个值，则要特殊处理，避免再添加进来

["381. Insert Delete GetRandom O(1) - Duplicates allowed" StructDesign]()

### "381. Insert Delete GetRandom O(1) - Duplicates allowed"

["381. Insert Delete GetRandom O(1) - Duplicates allowed" StructDesign]()

### "382. Linked List Random Node"

- 解法：ReservoirSampling
  - 思路：
    - 每次采样，对所有元素遍历一遍，利用 ReservoirSampling 取得结果值

["382. Linked List Random Node" Random]()

### "384. Shuffle an Array"

- 解法：Fisher-Yates Shuffle
  - 要点：
    - 要保留初始的数组以便`reset`返回
    - 要保留被随机后的数组，以便下次继续对其随机处理，否则由于随机种子被重置，无法真正实现随机，Case 会不过

["384. Shuffle an Array" Random Golang]()

### "395. Longest Substring with At Least K Repeating Characters"

- 考点：滑窗、常数内尝试
- 思路：
  - 从"子数组连续"这个特性，想到用 SlidingWindow
  - 控制窗内元素种类是个难点。由于字母集合总数有限，干脆尝试 1 ～ 26，共 26 次，不影响时间复杂度
  - 在滑窗过程中记录几个关键参数：每种字母个数、已包含字母种类、当前未达到 k 个的字母种类数
  - 在新增、删除元素时，维护好"当前未达到 k 个字母种类数"，然后当条件符合时更新最大长度结果

["395. Longest Substring with At Least K Repeating Characters" SlidingWindow Golang]()

### "398. Random Pick Index"

- 解法：ReservoirSampling
  - 思路：
    - 类似"382. Linked List Random Node"，需要对所有元素遍历一遍才可以保证概率相同
    - 只有指定的元素才参与随机，所以增加一个判断条件

["398. Random Pick Index" Random]()

### "403. Frog Jump"

- 解法：DP
  - 思路：用二维 map 或二维数组都可以
    - 记录青蛙能到达的下一个位置和本次跳跃步数，直到得出结果
  - 时间复杂度 O(n^2)，空间复杂度 O(n^2)

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

### "421. Maximum XOR of Two Numbers in an Array"

- 解法：XOR + HashMap
  - 思路：
    - 题目用双重循环很容易实现，但是要求 O(n)时间复杂度，需要进一步设计
    - 设最终找到的数为 x，那么 x 的高位二进制 1 越多结果越大，所以我们考虑按位从高到低找 1
    - 根据 XOR 运算特点有`x = a^b <=> a = x^b`
    - 假设`pre_k_x`为 x 的前 k 位，则同样有`pre_k_a = pre_k_x ^ pre_k_b`
    - 我们对每一位尝试查找是否存在 1，然后查找过程类似"1. Two Sum"，利用 HashMap 实现快速查找
  - 时间复杂度计算：O(nlogC)
    其中 n 是元素数量、C 是元素值的范围，本题是`(2^31) - 1`，由于`logC = 31`，所以最终时间复杂度符合`O(n)`要求

["421. Maximum XOR of Two Numbers in an Array"BinaryOperation Golang]()

### "435. Non-overlapping Intervals"

- 解法：Greedy
  - 思路：
    - 按照 end 时间将 interval 排序，我们想寻找尽量早结束的 interval 这样后面独立 interval 数量可能更多
    - 排序后遍历，如果 interval 是独立的，则更新 end 值，否则 count++

["435. Non-overlapping Intervals" Greedy Golang]()

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

### "470. Implement Rand10() Using Rand7()"

- 解法 1: 暴力采样，100 次采样时**不通过**

  - 思路：
    - 用 10 个`[1,7]`采样结果相加得到`[10,70]`，然后`/10`取得结果
  - 特点：
    - 时间复杂度固定 O(7n)->O(n)，但是不符合性能要求

- 解法 2: 拒绝采样，1000 次采样时**不通过**

  - 思路：
    - 采样两次求和，如果范围在`[2,11]`，则返回`[2,11]-1`，否则循环找到合适随机值
  - 特点：
    - 至少采样两次，并且有`3/7`概率丢弃，所以每次采样期望次数为`2*(4/7+(4/7)^2+...)≈6`，时间复杂度 O(6n)，还是不能符合性能要求

- 解法 3:
  - 思路：
    - 利用`[1,7]`找到最快接近`[1,10]`的方法，用`7*7 = 49`的方法，只需要两次`rand7()`采样就能找到最接近 40(10 的整数倍)目标
    - 保留 40 以内的有效值，期望采样次数为`2*(4/5+(4/5)^2...)≈2.4`，时间复杂度 O(2.4n)，符合性能要求
      ![470. Implement Rand10() Using Rand7()](/assets/images/2020-12-27-LeetCode_Question_Category_470.png)
  - 优化思路：
    - 上面`[41,49]`的随机值被抛弃了，实际上可以临时保留，下次凑够 10 的整数倍时还可以使用，这样用少量空间存储随机值，可以减少随机函数调用次数

["470. Implement Rand10() Using Rand7()" Random]()

### "474. Ones and Zeroes"

- 考点：
  - 零一背包+二维逆序推演
- 技巧：
  - 本题只求最大值，所以不用精确推算结果，在保证无**后效性**的同时，始终保持数组靠后为最大值即可

["474. Ones and Zeroes" DynamicPlanning]()

### "477. Total Hamming Distance"

- 解法：
  - 思路：
    - 首先要明白"汉明距离"的概念，是两个二进制数间异或后 1 的数量
    - 首先想到双循环，但是规模是`10^5`，`O(n^2)`无法满足要求
    - 想到先统计所有值每个位置为 1 的出现次数，然后遍历每个元素在这些位上的情况，统计出结果
    - 进一步优化，某个位置如果有 m 个元素为 1，则其他元素必然累加这个 m，可以对位做第一层循环、nums 做第二层循环
  - 时间复杂度：
    `O(n*L)`，其中`L==30`

["477. Total Hamming Distance" BinaryOperation]()

### "478. Generate Random Point in a Circle"

- 解法：RejectSampling
  - 思路：
    - 利用随机函数生成`[0,1)`之间的数，进而生成 x y 两个值，在`2r*2r`的正方形内，如果在圆内则采用，否则循环重试

["478. Generate Random Point in a Circle" Random]()

### "486. Predict the Winner"

- 解法：Minimax(dfs+memo)
  - 思路：
    - 每次可以取左边或右边，那么对手要取剩下的，剩下的范围比之前更小，所以可以利用 memo 记录
    - 用二维 memo 记录结果，每次 dfs 需要 left right 两个参数限定范围
    - 计算过程中只要记录两者之差会更容易，如果最后需要值，只需要根据两者之差和所有元素和计算二元一次方程即可

["486. Predict the Winner" Minimax]()

### "494. Target Sum"

- 考点：
  - 概率 DP、偏移重整
- 思路：
  - 尝试 1、2、3 个元素的组合，可发现状态转移公式
  - 利用`sum <= 1000`的特性，用`+1000`的方式规避负数下标出现

### "497. Random Point in Non-overlapping Rectangles"

- 解法：preSum + BinarySearch + Random
  - 思路：
    - 按照["528. Random Pick with Weight" Random]()的思路实现
    - 不同点是计算单元是一个矩形面上的点
    - 最后需要根据命中矩形同时根据偏移量求出坐标
  - 要点：
    - 要精确统计每个面上的点，包括边上的点，用公式：`count := (x2 - x1 + 1) * (y2 - y1 + 1)`
    - 计算命令矩形下标时，[二分查找模板](/2021/05/05/2021-05-05-BinarySearch/)中的`isLeft(m)`比较难匹配，改为`isRight(m)`匹配。
      注意点的统计是从`0`开始的，所以如果一个`2*2`的矩形，上面的点是`0 1 2 3`，所以`4`已经是`右边区域了`

["497. Random Point in Non-overlapping Rectangles" Random]()

### "518. Coin Change 2"

- 考点：
  - 概率 DP、完全背包版
- 思路：
  - 标准完全背包算法
- 注意：
  - 题目是统计"组合数量"，无需考虑"排列"，所以金币的不同位置不会影响结果，
    所以第一层循环必须是金币面额，这样可以避免金币位置的"后效性"

["518. Coin Change 2" DynamicPlanning Golang]()

### "523. Continuous Subarray Sum"

- 解法：prefix sum + math(同余定理) + HashMap
  - 如果两个数 x y 对某个 k 取余结果相同，那么这两个数的差值是 k 的整数倍

["523. Continuous Subarray Sum" Math]()

### "525. Contiguous Array"

- 解法：prefix sum + HashMap
  - 思路：
    - 利用 preFix 分别统计 1 和 0 的数量
    - 遍历中计算 1 和 0 的数量差，并记录在 HashMap，同时记录下标
    - 如果数量差再次出现，说明中间段的 1 和 0 增长相同，可记录一次结果
  - 改进：
    - 其实关键是 1 和 0 的数量差，所以只需要一个 preSum 值就可以，遇到 1 就`++`，遇到 0 就`--`

### "528. Random Pick with Weight"

- 考点：
  - 带权重的随机选择
- 解法：preSum + 二分查找
  - 利用 pre sum 计算随机用数组
  - 利用二分查找模板实现快速查找，用`isRight(m)`更容易实现

["528. Random Pick with Weight" Random]()

### "554. Brick Wall"

- 解法：HashMap
  - 思路：
    - 利用 HashMap 统计所有砖缝位置出现次数，去掉最左右两边的缝
    - 返回`行数 - 某缝位置最大次数`

### "560. Subarray Sum Equals K"

- 思路：
  - **子数组和**结合**累加数组**思想，可以把问题化简为`数组中 A - B = k <=> A - k = B`
  - 由于可以重复统计，并且统计结果数是递增的，可以利用 map 进行累加，`map[A-K]==count`
  - 起始处，要有`map[0]=1`，以便统计独立元素直接匹配的情况

["560. Subarray Sum Equals K" Golang]()

### "633. Sum of Square Numbers"

- 解法：math 分析 + 双指针
  - 思路：
    - `a^2+b^2==c`考虑用双指针法从左右逼近 a 和 b。
    - a、b 的范围中 a==0，`b^2`的最大值不会超过 c，所以`b <= sqrt(c)`
    - 由于`b^2`足够大，`a^2 <= (b^2-(b-1)^2)`，所以`right`只需单向运动，无需反复运动，`left`也是

["633. Sum of Square Numbers" Math Golang]()

### "645. Set Mismatch"

- 解法：In-Place 思想 + BucketSort
  - 思路：
    - 题目已经给了提示，先按 In-Place 思想找到重复元素，这时当前元素位置设置为 0，表示"空洞"
    - 然后继续利用 In-Place 遍历，直到所有元素归位
    - 最后顺序遍历一次，找到最终空洞位置

["645. Set Mismatch" Sort]()

### "664. Strange Printer"

- 解法 1: 范围 DP + DFS + mem
- 思路：
  - dp[i][j] 表示 i~j 范围最小打印的次数
  - 对所有范围递归求解，每个范围尝试每个点为分割点
  - 如果分割点和末尾字符一致，则末尾字符可以少打印一次，`dp[i][j] = min(dp[i][j], dfs(s, i, k) + dfs(s, k+1, j-1))`
  - 在递归开始要先对当前 dp 赋一个保底值

["664. Strange Printer" DP]()

### "677. Map Sum Pairs"

- 解法：TrieTree

### "688. Knight Probability in Chessboard"

- 解法：dp
  与之前的"935. Knight Dialer"原理相同，但是增加了两点复杂度：
  1. 可落子的区域变大，可能性更多，需要用两个大的棋盘来演化，并且统计数值会非常大。
  2. 答案要求计算概率而不是统计数值，由于统计数值过大，中间计算过程就会溢出(超出 double 型)，
     所以棋盘要存储每一位置的概率(double)，并且每次引用上一次演化结果时要"/8.0"(上一次位置到当前位置有 8 种可能结果)而不是累计到最后"/pow(8.0, K)"。

### "718. Maximum Length of Repeated Subarray"

- 解法 1：dp

  - 思路：
    - 类似"edit distance"，进行二维 DP
  - 时间复杂度：`O(M*N)`

- 解法 2：RollingHash + BinarySearch
  - 思路：
    - 由于重复子序列有递增特性，即如果有 3 个元素相同则必然有 2 个元素相同，所以可以利用二分查找猜子序列长度
    - 可以利用 RollingHash 生成子序列指纹
  - 时间复杂度：`O((N+M)*min(N,M))`

### "740. Delete and Earn"

- 解法：DP + Bucket
  - 思路：
    - 根据条件，每个元素值取后，附近的就没法取，这个和"198. House Robber"逻辑类似，可以 DP 解决
    - 由于元素范围在`1 <= nunms[i] <= 1e4`，可以用 Bucket 做统计，然后再 DP

["740. Delete and Earn" Bucket DP Golang]()

### "752. Open the Lock"

- 解法：BFS

["740. Delete and Earn" BreadthFirstSearch BFS]()

- 解法：`A*`
  - 思路：
    - 利用到目标点曼哈顿距离作为额外附加距离

["740. Delete and Earn" BreadthFirstSearch `A*`]()

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

### "773. Sliding Puzzle"

- 解法：BFS

- 改进解法：`A*`

### "781. Rabbits in Forest"

- 解法：HashTable
  - 思路：
    - 利用 HashMap 记录`map[单色兔子总数]已知兔子数量`
    - 每个兔子报数`对应的该颜色兔子数量 = 报数数 + 1`
    - 如果相同数量可以凑整，则`总数 += 完成凑整的兔子数量`
    - 最后统计所有未能凑整的兔子对应的数量和

### "787. Cheapest Flights Within K Stops"

- 解法：Bellmen-Ford
  - 思路：
    - 可以利用 Dijkstra 实现，但是用 Bellmen-Ford 正好可以利用 K 个中转站的特性

["787. Cheapest Flights Within K Stops" Graph]()

### "810. Chalkboard XOR Game"

- 解法：XOR + 博弈论
  - 规则：
    1. Alice 先选
    2. 轮到任何一名玩家选时，黑板上剩下当前所有数的异或值为 0 时，该玩家获胜
    3. 没有任何数字时，当前数组的异或值为 0
  - 思路：长度为偶数非常重要
    1. 假设此时偶数数组所有数的异或值为 0，那么 Alice 就直接获胜
    2. 如果此时偶数数组所有数的异或值不为 0。得到结论：
       至少有 2 个以上的数使其数组的异或值不为 0(记作 aa, bb)。
       Alice 会选这两个数中的一个，假设他选了 aa。此时 Bob 有两种方式：
       1. 选剩下的 bb, 那么此时数组长度又回到了偶数。
          情况一：此时数组异或值为 0，那么 Alice 获胜。
          情况二：数组异或值不为 0，那么又回到了步骤 2，这一步可以一直到数组的数被选完，然后轮到 Alice 选，空数组 Alice 获胜(规则三)。
       2. 如果 bob 不选 bb，那么轮到 Alice 选时，他也跳过 b，因为 bb 有可能是此时唯一的数选完后数组的异或值为 0。
    3. 我们知道偶数先选方是一定获胜的！
       那么对于 Bob 也是同理，如果 Alice 是奇数方，然后选去一个数字后数组变为偶数，那么 Bob 此时一定会获胜。
       那么此时 Alice 只有一种情况会赢，那就是在他第一次选的时候，黑板上所有数的异或值就已经为 0 了(规则二)。

["810. Chalkboard XOR Game" Math]()

### "818. Race Car"

- 考点：复杂问题建模

  - 已知：可以用 A 加速、R 减速到 0 并反转方向、target 是最终目标位置、求最少指令数
  - 令$A^k$表示连续 k 个 A，则在同方向走$2^k-1$步可表示为
    $$A^{k_0}RA^{k_1}...A{k_n}R$$
    这个表达式意思是经过一系列前进、后退动作最终到达了目标位置。
    最后面的 R 可以去掉(已经到达目的地，转向没有意义)，步数和方向可以用表达式表示为：
    $$(2^{k_0}-1)+(2^{k_2}-1)+...-(2^{k_1}-1)-...$$
    可以看出偶数次转向是向前的，奇数次转向是向后的
  - 设 K (大写)为 terget 的二进制最高位数，那么有
    $$target \leq 2^{K}-1$$
    这个表达式的物理意义是，如果已经超过了 target，那么就该停下向后开，否则冲出去太长的距离再往回折返肯定是不划算的

- 解法 1: Dijkstra

  - 思路：
    - 设位置为 i，每个位置可以最多加速${k_j}$次。根据上面分析可得${k_j} \leq K$，
      从 i 点可以向后加速${k_j}$次然后调头，每个加速和调头都会增加一次指令步数
    - 逐步将范围内所有位置的最小步数演化出来，最后返回结果，每个点可以走向 K+1 个点
    - 注意在没有 R(调头)时，到达此次演算目标的剩余距离为$i-(2^{k}-1)$，而调头后距离变为$(2^{k}-1)-i$表示反向走。
    - 由于最终结果的位置可能会被到达多次，而最后才直到最小距离，所以本 Dijkstra 算法会计算所有范围内的点。
      终止条件靠$[-(2^{k}-1), (2^{k}-1)]$范围控制
  - 时间复杂度：O(TlogT)，其中 T 是边界距离范围
  - 空间复杂度：O(T)
  - 优点：逻辑相对简单
  - 缺点：由于本题利用 map 作为存储，内存占用较多

["818. Race Car" Dijkstra Golang]()

- 解法 2: BFS + memory

  - 把"位置+速度"作为一个基本单元，放在 queue 里进行 BFS，每次遍历尝试继续加速或减速
  - 由于规模较大"10000"，所以要进行 memory 剪枝，以"位置+速度"作为 key 保存在 map 中
    - 由于规模在`1~10000`之内，同时速度是"2^n"，可以用一个 32 位表示一个 key"位置+速度+速度方向"，
      但是追求极致节省内存没必要，可以用一个结构体或 string 表示一个 key
  - 另外还要限制 BFS 范围：当`范围 > 2*target`时，就不再添加元素
    - 到达最终位置的路径可以是一个"蛇形"过程，超出太远就没有意义了

- 解法 3: DP
  - 思路：
    - 令 dp[t]表示走到 t 位置用的最小步数，要走到一个 t 有三种情况：
    - 1. 如果$t=2^{k}-1$，则刚好走 k 步
    - 2. 永远不超过 t，向右走到某个位置，然后向左走 j 步，再向右走到头($2^{k-1}-1$处)。
         行走路径为$XRA^{j}RA^{k-1}R$，其中 X 表示到某位置的路径，由 dp 的演化顺序保证，最后的 R 表示停下来。
         花费步数为
         $$dp[t-2^{k-1}+2^{j}]+(k-1)+(j-1)+3$$
         上面`+3`表示一共三个 R
    - 3. 超过 t 然后往回走，$A^{k}RX$，走到$2^{k}-1$后往回走的距离为$(2^{k}-1)-t$，步数为
         $$dp[(2^{k}-1)-t]+k+1$$
    - 上面 2、3 的 dp 值都可以取历史最小值
  - 时间复杂度：O(TlogT)，对于每一个位置 x 要循环 O(logx)次
  - 空间复杂度：O(T)
  - 优点：内存使用效率高，性能较高
  - 缺点：逻辑较复杂

["818. Race Car" DP Golang]()

### "879. Profitable Schemes"

- 解法：零一背包 + 三维 DP
  - 思路：
    - 本题可以看作是零一背包：人数是背包容量、工作分组是物品体积、分组盈利是物品重量
    - 零一背包可以用二维 DP 解决，这里需要考虑第三维：是否达到了 minProfit，如果达到了才累计到结果
    - 所以用三维 DP 解决，三个维度分别是：i 分组选择(物品)、j 人数(容量状态)、k 需满足最底盈利，值表示可达到的计划数
    - 关键状态转移方程`dp[i+1][j][k] = (dp[i][j][k] + dp[i][j-num][max(0, k-profit[i])]) % MOD`
      - 如果新增人员可以满足新的组需要的人数，则在之前的基础上新增盈利，计算盈利时要先减掉本组所需人数
      - 如果新增人员无法满足新组要求人数，则继续保持之前的结果`dp[i+1][j][k] = dp[i][j][k]`
  - 要点：
    - 最后返回值时要对每个人员情况下的结果累加
    - 在处理过程中要做 MOD 处理
  - 改进：
    - 可以看到三维中`i`这个维度只会用到上一层，并且也只可能直接复制。所以可以去掉这个维度
    - 由于去掉了上面维度，要给每个第二维开头加个 1
    - 为了避免人数 j 和最底盈利 k 的后效性，要逆序遍历

["879. Profitable Schemes" DynamicPlanning]()

### "909. Snakes and Ladders"

- 解法：BFS
  - 思路：
    - 每步可以有 6 个位置选择，按规则跳转
  - 要点：
    - 由于棋盘的蛇形排列，遍历下一位置需要特殊逻辑处理，可以先简化为最简单的行、列取值，
      然后再根据规则进一步计算，这样逻辑复杂度会低很多，如下：
      ```Go
      getCoordVal := func(pos int) (int) {
              x := (pos-1)/n
              y := (pos-1)%n
              if x&1 > 0 {
                  y = n-1-y
              }
              x = n-1-x
              return board[x][y]
          }
      ```
  - 注意：
    - 由于本题存在"回退"的路径，所以无法使用`A*`算法，否则第一个找到的最小步数未必是答案

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

### "913. Cat and Mouse"

- 解法：Minimax

["913. Cat and Mouse" Minimax C++]()

### "918. Maximum Sum Circular Subarray"

下面这种解法时间和空间性能都较差，但是易于理解，以后再考虑学习性能较好的解法。

1. 把循环数组想象两个重复的数组拼接在一起，按照单个数组的解法，计算一个 `2*N+1` 长度的前 N 项和数组。
2. 利用一个 deque 存储可用于被减的前面的下标，初值插入 0。
3. 从 1~2N 开始遍历，首先去除间隔距离过远的下标，然后计算当前值与 dq 最前面的下标对应的值相减结果。
4. 将当前下标插入后边前，先将队尾所有值小于当前的弹出。
5. 总体思路是保持 deque 中有合适的比较值的下标，要么下标合适、要么值要够小(被后边减时结果会更大)。

### "930. Binary Subarrays With Sum"

- 解法：PrefixSum + HashMap
  - 思路：
    - 通过 PrefixSum 可以实现对任意字串的和的计算
    - 然后利用"Tow sum"的方法滚动计算符合条件的字串
    - 由于 0 不会影响 sum 结果，所以需要重复统计，需要用 HashMap 对重复 presum 元素进行累加

["930. Binary Subarrays With Sum" HashTable]()

### "935. Knight Dialer"

- 解法：
  此题看似有数学规律，实际只能靠基本演化逻辑，通过动态规划推导出最后结果。
  每次演化都要从之前的某个位置过来，所以只需要两组位置计数不断演化即可。

### "974. Subarray Sums Divisible by K"

- 考点：
  - 连续整数组成的子序列，可以利用 sum(i,j)来快速表达
  - Math: **同余定理**，`(a-b)%m == 0 <=> a%m == b%m == k`，如果两数之差可以被 m 整除，那么两数分别对 m 取余的值相同
  - 利用 HashTable 对已存在元素累加
- 思路：
  - 设 P[i] 是前面 i 个数累加和，`sum(i,j) == P[j]-P[i-1]`
  - 根据同余定理`sum(i,j)%K == 0 <=> P[j]%K == P[i-1]%K`

["974. Subarray Sums Divisible by K" HashTable Golang]()

### "981. Time Based Key-Value Store"

- 解法：HashMap + BinarySearch

### "986. Interval List Intersections"

- 解法 1: LineSweep
  - 思路：
    - 影响区域重叠的只有 interval start end 两个位置点
    - 可以将位置点拆分、排序，然后顺序遍历统计当前存在的线段数，如果 >1 则记录
    - 注意排序时先处理相同坐标点的 end，这样能统计到更大的数

["986. Interval List Intersections" LineSweep Golang version1]()

- 解法 2: LineSweep + SlidingWindow
  - 思路：
    - 由于原始数据保证顺序、本数组不重叠，可以利用这个特性直接扫描
    - 利用 max(start) 和 min(end) 函数尽量找到当前重叠区域
    - 保持当前区域为可能产生交集的 Window，逐步移动两个数组的下标

["986. Interval List Intersections" LineSweep Golang version2]()

### "993. Cousins in Binary Tree"

- 解法：
  - 思路：DFS
    - 利用二叉树的特征，在查找过程中限定各种边界条件来减少无用搜索
    - 每次起始递归函数前都要初始化全局变量
    - 如果已经找到了目标层，则不再向更深遍历
    - 确保同层并且父结点不同

["993. Cousins in Binary Tree" Tree Golang]()

### "1004. Max Consecutive Ones III"

- 思路：
  关键字`contiguous`，所以考虑用 sliding window

### "1006. Clumsy Factorial"

- 思路：
  利用 stack 进行无括号的四则运算，乘除直接计算、加减入栈，最后累加栈中所有元素

### "1011. Capacity To Ship Packages Within D Days"

- 解法：BinarySearch
  - 思路：
    - 初看题目，以为是背包问题。但是题目要求货物按顺序放置，所以无法背包。
    - 题目可以转化为二分查找，在范围内尝试船的容量
    - 二分查找的最小值是最大物品重量、最大值是全部物品重量和
  - 要点：
    - Go 可以利用`sort.Search(int, func(int) bool)`实现 LowerBound 二分查找
    - 注意货物不能切割，所以如果当前放不下就要放到第二天

["1011. Capacity To Ship Packages Within D Days" BinarySearch Golang]()

### "1035. Uncrossed Lines"

- 解法：DP
  - 思路：
    - 经过逻辑推导，可以发现逻辑和"1143. Longest Common Subsequence"一样，求出最长公共子序列

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

### "1039. Minimum Score Triangulation of Polygon"

- 解法：DFS+Memo
  - 思路：用区间 DP DFS+Memo 模板解决

### "1044. Longest Duplicate Substring"

- 解法：RollingHash + BinarySearch
  - 思路：
    - 由于计算规模较大`3*(10^4)`，所以无法用`O(n^3)`的三重循环暴力匹配
    - 假定字串长度为 K，则题目转化为"是否有长度为 K 的重复字串"
    - 利用 RollingHash 可以快速进行字符串字串匹配
    - 假设某个 K 符合要求，那么根据字串字串的特点 K-1 也必然符合要求，可以利用 BinarySearch 查找这个边界长度 K
    - 用完整比较的方法解决 Collision
  - 时间复杂度：`O(nlogn)`

["1044. Longest Duplicate Substring" SlidingWindow RollingHash]()

### "1049. Last Stone Weight II"

- 解法：0-1 背包 + 题目分析
  - 思路：
    - 题目参照"416. Partition Equal Subset Sum"，进行破题
    - 只要有两块石头，那么两两就可以抵消，最终一定能剩下一块石头。
      在抵消过程中，我们希望尽量能抵消更多重量。可以想象将石头分为两堆，这两堆之间相互抵消，最后留下一块，正好是两堆重量差。
      所以这道题可以转化为 0-1 背包：如何找到一堆石头，总重量尽量接近全部重量的一半，这样最终抵消后剩下的最少。
  - 要点：
    - 本题的目标是找"最值"而不是"恰好放下"，所以应使用不一定能填满的 DP 模板

["1049. Last Stone Weight II" DynamicPlanning Golang]()

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

### "1074. Number of Submatrices That Sum to Target"

- 解法：二维 preSum + hash + matrix 遍历
  - 思路：
    - 这道题的基本思路先要会："560. Subarray Sum Equals K"
    - 在 matrix 中对每列进行 preSum 压缩成一维，然后用上面现成函数
    - 注意对 matrix 的遍历至少要选取两行，所以时间复杂度是`O((m^2)*n)`

### "1140. Stone Game II"

- 解法：Minimax
  - 思路：
    - 利用 dfs+memo 模板实现
    - 利用 postfixSum 可以实现快速计算区间值
    - 由于模板计算过程使用差值，最后结果需要解二元一次方程
    - 由于本题是求尽量大的值，中间值可能为负，所以初始值要为`math.MinInt32`

["1140. Stone Game II" Minimax]()

### "1143. Longest Common Subsequence"

- 解法：DP
  - 这是一个二维 DP，类似"edit distance"，分别以两个字符串长度作为 dp 方向
  - 设`dp[i][j]`表示`text1[0..i]`和`text2[0..j]`的最长公共子序列
  - 边界条件: `i/j == 0`时，由于一方没有字符串，所以最长公共子序列长度必然为 0
  - 当`text1[i] == text2[j]`时，`dp[i][j] = dp[i-1][j-1] + 1`，表示存在相同字符，长度增 1
  - 当`text1[i] != text2[j]`时，`dp[i][j] = max(dp[i-1][j], dp[i][j-1])`，表示延续历史最大长度

["1143. Longest Common Subsequence" DP Golang]()

### "1192. Critical Connections in a Network"

- 解法：Tarjan
  - 思路：
    - 基于基本的 Tarjan 算法
    - 利用回溯时，`LOW_next < DFN_i`的特性，确认环内点；`LOW_next == DFN_i`为环起始点；`LOW_next > DFN_i`为环起始点前面的点
    - 然后记录入环点和入环前的点组成的边
  - 注意：
    - 本题是双向联通图，所以从任意一点开始 dfs 都可以得到全部结果

["1192. Critical Connections in a Network" Graph]()

### "1128. Number of Equivalent Domino Pairs"

- 思路：
  - 只需要对"相同"的元素累加统计即可
  - 由于每个分量值`<9`，可以用一个排过序的两位数表示
  - 可以利用一个 100 个元素的 bucket 统计，这样就无需排序一次遍历就统计出来

### "1229. Meeting Scheduler"

- 解法：BF
  - 思路：
    - 这道题需要用到自动有序数据结构，并且最好容易得到前后元素，C++的 set(红黑树)就满足条件
    - 每次向前、后中较近的 interval 合并

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

### "1239. Maximum Length of a Concatenated String with Unique Characters"

- 解法：Backtracking + bit
  - 思路：
    - 每个可选字符串字母数量不超过 26，考虑用二进制
    - 先考虑用零一背包实现，由于是位的组合，需要用二维零一背包，
      但是考虑样本空间太大，占用超量存储，所以考虑用 map
    - 由于总规模较小，直接用 Backtracking 解决

["1239. Maximum Length of a Concatenated String with Unique Characters" Backtracking]()

### "1269. Number of Ways to Stay in the Same Place After Some Steps"

- 解法：DP + 规模边界条件
  - 思路：
    - 按照 DP 演化思路，有两个数组分别表示两步，逐步演化
  - 要点：
    - 按基本思路写完会超时，原因是`arrLen <= 10^6`每步演化的数组过大，
      而`steps <= 500`步数很小，`arrLen >= 251`之后的不会影响结果了，
      所以只要在演化前对`arrLen`进行一次范围修正即可

### "1288. Remove Covered Intervals"

- 解法：LineSweep
  - 思路：
    - 要保证两边都被覆盖的 interval 才删除
    - 我们排序时保证按 start 顺序排序，然后按 end 逆序排序
    - 之后遍历时候因为按 start 遍历，只要关注 end，一旦 end 被覆盖，说明 start 必然被覆盖

["1288. Remove Covered Intervals" LineSweep Golang]()

### "1310. XOR Queries of a Subarray"

- 解法：preSum + XOR
  - 思路：
    - 先求出异或的 preSum 数组
    - 然后利用 preSum 求出任意范围内的异或结果

["1310. XOR Queries of a Subarray" BinaryOperation]()

### "1316. Distinct Echo Substrings"

- 解法：RollingHash + PrefixSum

### "1392. Longest Happy Prefix"

- 解法：RollingHash
  - 思路：
    - 是否有 HappyPrefix 没有线性演化关系，无法用 BinarySearch
    - 从前向后取 hashPre，同时从后向前取 hashAfter

["1392. Longest Happy Prefix" SlidingWindow RollingHash]()

### "1406. Stone Game III"

- 解法：Minimax + postfixSum
- 思路：
  - 利用 Minimax 模板计算差值即可
  - 注意本题在末端需要取得最后一个元素，所以 postfixSum 的容量要+1

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

### "1442. Count Triplets That Can Form Two Arrays of Equal XOR"

- 解法：XOR + preSum + HashTable + 推导优化
  - 思路：
    - 最直观的方法是求出 preSum 数组，然后分别遍历 i j k 三个坐标计算结果，时间复杂度 O(n^3)。
      但是这种方法太初级，经过优化可以达到 O(n)时间复杂度：
    - 设前 i 项的 preSum 值是 Si，preSum 的数组元素是 arri。根据 XOR 特性有：
      `Si ^ Sj = arr(i) ^ ... ^ arr(j-1)`，那么`[i,j]`异或和为`S(i) ^ S(j+1)`，
      题目中左右两段异或和 a b 可以表示为`a = S(i) ^ S(j)`、`b = S(j) ^ S(k+1)`，
      如果`a == b`则有`S(i) ^ S(j) == S(j) ^ S(k+1)`根据 XOR 的交换律，得出`S(i) == S(k+1)`，
      也就是前后有相同的 preSum 值，则找到了目标 tuple，中间的任何 j 都成立(遍历降低一维)。
  - 设当前处理到 k 时，前面有 m 个 i(i1 i2 ... im)都满足`S(i) == S(k+1)`，则这时它们对结果的贡献数量为：
    `(k-i1) + (k-i2) + ... + (k-im) = m*k - (i1+i2+...+im)`，
    所以在遍历过程中我们要记录满足`S(i) == S(k+1)`的 i 出现次数**count**和下标 i 之和**total**，
    可以利用 HashTable 记录这两个值

["1442. Count Triplets That Can Form Two Arrays of Equal XOR" BinaryOperation]()

### "1449. Form Largest Integer With Digits That Add up to Target"

- 解法：完全背包 + 字符串(itoa)
  - 思路：
    - cost 是物品体积、index 是物品值、target 是背包容量
    - 两层循环，第一层是背包容量状态、第二层是物品
    - 最后将结果生成字符串形式返回

### "1463. Cherry Pickup II"

- 考点：
  - DP
- 思路：
  - 用三维数组 dp[x][y1][y2]表示某行下机器人 1、机器人 2 的位置下已获取的最大分数
  - 转移方程：`dp[x][y1][y2] = max(dp[x][y1][y2], dp[x-1][y1-1~y1+1][y2-1~y2+1])`
  - 注意设置初始行，然后开始迭代。考虑到初始行可能是 0 值，所以 dp 初始值设置为无效值 0
  - 最后需遍历找到最后一行中的最大值返回，注意需要对 y1、y2 两个变量进行遍历
  - 优化：可以只用两行数据进行迭代

### "1482. Minimum Number of Days to Make m Bouquets"

- 解法：BinarySearch
  - 思路：
    - days 范围较大，但是具有顺序性，即一旦某个 days 满足条件，那>=它的都满足，所以可以利用二分查找
    - 在某个 days 下，是否符合条件可以简单遍历统计实现
  - 时间复杂度 O(mlogn)

### "1510. Stone Game IV"

- 解法：Minimax
  - 思路：
    - 只要对手能符合条件，则不能必胜，因此需要对手必输的情况下，自己才能必胜
    - 按 Minimax 模板即可

### "1547. Minimum Cost to Cut a Stick"

- 解法：区间 DP 模板 + 边界逻辑
  - 思路：
    - 可以直接用区间 DP 解决，但是效率较低。其实可以利用`cuts`数组长度较小的特性，只处理有效切割点。
    - `dfs(int l, int r)`函数表示`[l,r)`左闭右开区间结果的计算
    - 注意 cost 的计算需要根据当前是否为边界、curts 点来计算

["1547. Minimum Cost to Cut a Stick" DynamicPlanning]()

### "1584. Min Cost to Connect All Points"

- 解法：MST Kruskal
  - 思路：
    - 由于本题是一个**全联通图**，即任意一点可以连到其他点，所以使用 Prim PQ 算法可能超时
    - 用 Kruskal 可以勉强通过
    - 用 Prim Naive 可以较短时间通过

["1584. Min Cost to Connect All Points" MST Golang]()

### "1649. Create Sorted Array through Instructions"

- 解法：BIT + bucket
  - 思路：
    - 利用 BIT 当成 bucket，记录每个元素出现的次数，然后取当前元素前后区间内元素出现次数总和综合统计
  - 要点：
    - 原本 BIT 的容量是用于存放下标的，本题是存放值，所以范围要+1，再加上模板的+1，最重要+2

["1649. Create Sorted Array through Instructions" BinaryIndexedTree Golang]()

### "1707. Maximum XOR With an Element From Array"

- 解法 1: XOR + TrieTree + DP

  - 思路：
    - 假设给定了集合 nums 不限制 mi，求 xi 的过程可以看作从左向右查看二进制位匹配的过程，
      由于 XOR 运算，最好的结果是两位不同，可以把原始 nums 每个元素放入二进制树的 TrieTree，然后遍历取得最大值
    - 上面的 TrieTree 的构建可以类似 DP 从小到大，也就是 nums 排序后逐步构建，这样只要对 queries 的 mi 排序，
      就能逐步求得结果，注意要记录 queries 的元素下标以组成答案
    - 特殊点：如果没有满足 mi 的 nums 元素，则结果为 -1
  - 时间复杂度：`O(NlogN + QlogQ + (N+Q)*L)`
    其中 N 是 nums 规模；Q 是 queries 规模；L 是元素的二进制长度 30 位
  - 优化算法：
    上面算法较耗时部分是左边两个排序导致的`nlogn`，能否不排序？
    可以在 TrieTree 构建时，每个结点记录孩子结点的最小值，那么根结点会记录所有结点的最小值。
    这样可以直接识别出返回值为-1 的 mi

["1707. Maximum XOR With an Element From Array" TrieTree]()

### "1711. Count Good Meals"

- 解法：HashMap
  - 思路：
    - 类似"three sum"，遍历时利用 hashTable 结构快速判定是否有历史匹配元素
    - 由于不同下标认为是不同元素，所以需要用`map[int]int`
    - 遍历到每个元素，都遍历`1~2^22`，这些 2 的幂

### "1720. Decode XORed Array"

- 解法：异或特性
  - 思路：
    - 异或运算具有自反性，`a^b^a == b`
    - 由于我们知道了`first`，可以利用其特性求出`ret[1] = first^encoded[0]`
    - 再用新求出的值迭代取得所有结果`ret[i+1] = ret[i] ^ encoded[i]`

["1720. Decode XORed Array"BinaryOperation Go]()

### "1723. Find Minimum Time to Finish All Jobs"

- 解法：BinarySearch
  - 思路
    - 如何分配任务难以直接获取最优解，考虑用 backtracking，但是先要确定每人最大负荷
    - 最大负荷有个特点，一旦某个负荷满足，那么更大的一定也满足，所以可以用二分查找
    - 二分查找的最小值是最大的单个任务，最大值是全部任务之和
    - 确定了某个单人最大负荷 limit 后，需要对每个商品进行 backtracking 处理，尽量把该商品尝试放在靠前的人身上
    - 在处理时，优先尝试较大的，所以要先对数组倒序排序
    - 如果某个人放某个物品已经放不下，则尝试下一个人
    - 有两个提前退出条件：
      - 如果当前人刚好以 limit 放满而之后的没有成功，说明一定不会成功，返回 false
      - 如果当前人是空的，而之后的人也一定是空的，这时如果失败了，说明后边的人也会失败，所以返回 false

["1723. Find Minimum Time to Finish All Jobs" BinarySearch]()

### "1734. Decode XORed Permutation"

- 解法：XOR + 逻辑
  - 思路：
    - 这道题是基于"1720. Decode XORed Array"的算法，只要求出`first`就可以推出结果
    - 题目条件**数组元素是最前面的 n 个正整数，n 必定是奇数**，我们假设为`A B C D E`
    - 设`AB`表示`A^B`，根据异或的结合律，我们可以求出一个"蕴含"了所有元素的值`ABCDE`
    - 根据已知的 encoded 数组`AB BC CD DE`，我们只要从中取出`BC DE`然后求得`BCDE`
    - 再根据异或的"自反"性`A = ABCDE ^ BCDE`，求得`first`
    - 然后根据"1720. Decode XORed Array"的算法求得结果即可

["1734. Decode XORed Permutation"BinaryOperation Go]()

### "1738. Find Kth Largest XOR Coordinate Value"

- 解法：XOR + preSum + nth_element
  - 思路：
    - XOR 的处理和`+`类似，可以适用于 preSum
    - 利用二维 preSum 可以求得 sum 矩阵
    - 然后遍历 sum 矩阵，将元素加入 array，通过排序取出目标值
  - 改进：
    - 二维 preSum 可以利用原 matrix 节省空间
    - 二维 preSum 可以一次遍历就完成，节省循环次数
    - 可以利用 DP 实现 preSum 一次遍历，转移方程：`dp[i][j] = dp[i-1][j]^dp[i][j-1]^dp[i-1][j-1]^matrix[i-1][j-1]`
    - "查找第 K 大元素"可以利用 nth_elemet 实现，时间复杂度 O(n)

["1738. Find Kth Largest XOR Coordinate Value" BinaryOperation]()

### "1818. Minimum Absolute Sum Difference"

- 解法：
  - 思路：
    - 想要尽量降低总 diff，那么需要尝试所有元素对，找到可能的最大改变量
    - 对于每一个 nums2，可以在 nums1 中找到最接近的两个元素进行尝试，如果 nums1 有序，则可以利用二分查找找到这两个最近的 nums1 元素
  - 步骤：
    1. 利用排序将 nums1 排序，可以单独将 nums1 复制达到一个数组中排序，也可以绑定 nums1 nums2 一起排序，这样可以节省容量
    2. 遍历 nums2 数组，同时利用二分查找计算 nums1 中最接近 nums2 的两个数，结合 nums2 下标对应的 nums1 中的数，计算最大减小量
  - 时间复杂度：O(nlogn)，空间复杂度：O(1)
  - 其他：
    - 利用双指针归并排序的思路，可以在步骤 2 中将时间复杂度降为 O(n)，但是由于步骤 1 已经达到 O(nlogn)，所以没有必要做这个优化

### "1833. Maximum Ice Cream Bars"

- 解法：Bucket Sort
  - 思路：
    - 由于 cost 范围有限(1 ～ 1e5)，可以用 bucket 排序后统计结果

["1738. Find Kth Largest XOR Coordinate Value" Sort]()

### "1846. Maximum Element After Decreasing and Rearranging"

- 解法：CountSort + Greedy
  - 思路：
    - 根据题目描述，可以想象为排序之后进行铺楼梯，并且当前可用砖块只能降低不能升高
    - 首先想到可以排序+遍历的方式填补空缺，最终找到最高点
    - 由于本题的特点，台阶必须连续，所以最高值不会超过 len(arr)，可以利用 Count Sort 进行排序

["1846. Maximum Element After Decreasing and Rearranging" Sort]()

### "LCP 07. 传递信息"

- 解法：DP
  - 思路：
    - 用一般的 DP 迭代演化即可
    - 注意题目给定的 Graph 结构无需改造，直接在每轮迭代中利用边即可
