---
layout: post
title: "LeetCode Question Category"
date: 2020-12-27 08:00:00 +0800
tags: Algorithm Leetcode
---

![LeetCode](/assets/images/2020-12-27-LeetCode_Category_1.jpg)
记录 LeetCode 题型分类及解题思路

# Array

### "31. Next Permutation"

- 原理：
  顺序排列的数组没有更小的值，倒序排列的数组没有更大的值。
- 解法：
  1. 从右向左查找第一个发生递减的位置。
  2. 从此位置向右查找最右边比当前值高一点的位置。
  3. 交换这两个位置。
  4. 将发生递减位置右边所有位置反转，呈现正序排列。

["31. Next Permutation"]()

# stack

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

# HashTable

### "49. Group Anagrams"

- 思路：
  利用 Anagram(异位词)特性，统计每个词的字母出现次数，生成一个"指纹"，然后通过 HashMap 收集相同指纹的词

["49. Group Anagrams"]()

# Binary Search

### "33. Search in Rotated Sorted Array"

- 思路：
  这道题是在二分查找的基础上，增加一层排除法逻辑，使得每次都能缩小一半查找范围。

["33. Search in Rotated Sorted Array"]()

### "34. Find First and Last Position of Element in Sorted Array"

- 思路：
  这道题是在二分查找的基础上，进一步利用算法细节特性："二分查找的结果是元素第一次出现的下标"。
  所以只需要再查询`按排序下一个元素出现的位置-1`就可以得到末尾位置

["34. Find First and Last Position of Element in Sorted Array" Golang]()

# Brute Force

### "42. Trapping Rain Water"

- 思路：
  假设每一个下标对应一个水柱，则水柱的高度由此位置最左和最右 bar 的高度综合决定。
  1. 从左向右遍历一遍，记录每个下标对应的左边最高的 bar 高度
  2. 从右向左遍历一遍，一边遍历一边计算累加当前位置的水柱高度，`水柱高度 = min(left, right) * height`

### "64. Minimum Path Sum"

- 思路：这道题有较好的 DP 解决方案，所以用 DFS 时性能相比较差就会超时。解决思路：从右下角开始动态规划，计算每个下一个位置可能的最小值。利用原空间就可以完整 DP 过程。

### "72. Edit Distance"

- 编辑距离在机器翻译、语音识别等数据科学领域经常用到。
- 动态规划解法：
  1. 用 dp[i][j]代表 word1[i]到 word2[j]的编辑距离，从空字符串开始演化，所以二维矩阵的范围要大 1。
  2. 初始化第 0 行和第 0 列，然后遍历计算 dp[i][j]。转移公式是：dp[i][j] = min(dp[i-1][j]+1, min(dp[i][j-1]+1, dp[i-1][j-1] + (word1[i] == word2[j]? 0, 1)))。
  3. 最后返回 dp[m][n]。

### "78. Subsets"

- 迭代法：
  可以将迭代过程看做：`[[]] -> [[],[1]] -> [[],[1],[2],[1,2]] -> [[],[1],[2],[1,2],[3],[1,3],[2,3],[1,2,3]]`
  每次将前面的内容复制到结尾，并将新的数字加入进去。

["78. Subsets" C++]()

- 回朔法：
  设置一个集合切片，每一个元素可以选取或不选取

["78. Subsets" Golang]()

### "84. Largest Rectangle in Histogram"

- 这道题的 stack 解题思路（时间复杂度 O(n)）很重要，在很多其他题中都会用到。
- 基本原理：
  1. 从左向右遍历，并不断将高度值的下标 push 入 stack。
  2. 当某一位置 height 值缩小时，那么之后就很难计算前面的面积了，所以就要把前面比当前值高的 stack 出栈，并计算前面的面积。
  3. 注意，计算面积时，宽度要当前下标和前面的"背影"相减，所以栈底是-1。

["84. Largest Rectangle in Histogram" Golang]()

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
