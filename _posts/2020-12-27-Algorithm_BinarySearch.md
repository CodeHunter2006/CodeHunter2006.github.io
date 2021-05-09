---
layout: post
title: "Algorithm Binary Search"
date: 2020-12-27 09:00:00 +0800
tags: Algorithm Leetcode
---

记录 Binary Search 的算法实现

### "33. Search in Rotated Sorted Array" Golang

```Go
func search(nums []int, target int) (ret int) {
    if len(nums) == 0 {
        return -1
    }
    //defer func(){ fmt.Println(nums[0], nums[len(nums)-1], ret) }()
    mid := len(nums)>>1

    if nums[0] <= nums[len(nums)-1] {
        if target > nums[len(nums)-1] || target < nums[0] {
            return -1
        } else if target == nums[mid] {
            return mid
        }
    }

    if ret = search(nums[:mid], target); ret != -1 {
        return ret
    } else if ret = search(nums[mid:], target); ret != -1 {
        return mid + ret
    }

    return -1
}
```

### "34. Find First and Last Position of Element in Sorted Array" Golang

- 可以利用`sort.SearchInts(a []int, x int) int`，用二分查找找到元素第一次出现的下标，如果没有找到则返回元素插入操作的下标

```Go
func searchRange(nums []int, target int) []int {
    l := sort.SearchInts(nums, target)
    if l == len(nums) || nums[l] != target {
        return []int{-1, -1}
    }
    r := sort.SearchInts(nums, target + 1) - 1
    return []int{l, r}
}
```

### "81. Search in Rotated Sorted Array II"

```Go
func search(nums []int, target int) bool {
    n := len(nums)
    if n == 1 {
        return nums[0] == target
    }
    for l, r := 0, n-1; l <= r; {
        mid := (l+r)>>1
        if nums[mid] == target {
            return true
        } else if nums[l] == nums[mid] && nums[mid] == nums[r] {
            l, r = l+1, r-1
        } else if nums[l] <= nums[mid] {
            if nums[l] <= target && target < nums[mid] {
                r = mid-1
            } else {
                l = mid+1
            }
        } else {
            if nums[mid] < target && target <= nums[r] {
                l = mid+1
            } else {
                r = mid-1
            }
        }
    }
    return false
}
```

### "300. Longest Increasing Subsequence"

```Go
func lengthOfLIS(nums []int) (ret int) {
    var dp []int
    for _, n := range nums {
        if idx := sort.SearchInts(dp, n); idx == len(dp) {
            dp = append(dp, n)
        } else {
            dp[idx] = n
        }
    }
    return len(dp)
}
```

```C++
int lengthOfLIS(vector<int>& nums) {
	vector<int> dp;
    for (auto i : nums) {
        auto it = lower_bound(dp.begin(), dp.end(), i);
        if (it == dp.end()) dp.push_back(i);
        else *it = i;
    }
    return dp.size();
}
```

### "1011. Capacity To Ship Packages Within D Days"

```Go
func shipWithinDays(weights []int, D int) int {
    // 查找重量的左右边界
    left, right := 0, 0
    for _, w := range weights {
        if w > left {
            left = w
        }
        right += w
    }
    return left + sort.Search(right-left, func(x int) bool {
        x += left
        day, sum := 1, 0
        for _, w := range weights {
            if sum + w > x {
                day++
                sum = 0
            }
            sum += w
        }
        return day <= D
    })
}
```

### "1723. Find Minimum Time to Finish All Jobs"

```Go
func minimumTimeRequired(jobs []int, k int) int {
    sort.Sort(sort.Reverse(sort.IntSlice(jobs)))
    l, r := jobs[0], 0
    for _, v := range jobs {
        r += v
    }

    return l + sort.Search(r-l, func(limit int) bool {
        limit += l
        sum := make([]int, k)

        var backtrack func(int) bool
        backtrack = func(idx int) bool {
            if idx == len(jobs) {
                return true
            }
            for i := range sum {
                if sum[i] + jobs[idx] <= limit {
                    sum[i] += jobs[idx]
                    if backtrack(idx+1) {
                        return true
                    }
                    sum[i] -= jobs[idx]
                }
                if sum[i]+jobs[idx]==limit || sum[i]==0 {
                    return false
                }
            }
            return false
        }
        return backtrack(0)
    })
}
```
