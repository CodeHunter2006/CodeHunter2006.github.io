---
layout: post
title: "LeetCode Category"
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
