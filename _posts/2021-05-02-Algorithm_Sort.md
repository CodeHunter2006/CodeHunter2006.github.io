---
layout: post
title: "Algorithm Sort"
date: 2021-05-02 09:00:00 +0800
tags: Algorithm Leetcode
---

记录 Sort 的算法实现

### "324. Wiggle Sort II"

```C++
int n = nums.size();
// Find a median.
auto midptr = nums.begin() + n / 2;
nth_element(nums.begin(), midptr, nums.end());
int mid = *midptr;
// Index-rewiring.
#define A(i) nums[(1+2*(i)) % (n|1)]
// 3-way-partition-to-wiggly in O(n) time with O(1) space.
int i = 0, j = 0, k = n - 1;
while (j <= k) {
    if (A(j) > mid)
        swap(A(i++), A(j++));
    else if (A(j) < mid)
        swap(A(j), A(k--));
    else
        j++;
}
```
