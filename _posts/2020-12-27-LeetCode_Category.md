---
layout: post
title: "LeetCode Category"
date: 2020-12-27 09:00:00 +0800
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

# HashTable

### "49. Group Anagrams"

- 思路：
  利用 Anagram(异位词)特性，统计每个词的字母出现次数，生成一个"指纹"，然后通过 HashMap 收集相同指纹的词

["49. Group Anagrams"]()
