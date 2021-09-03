---
layout: post
title: "Algorithm Array"
date: 2020-12-27 09:00:00 +0800
tags: Algorithm Leetcode
---

记录 Array 的算法实现

# BinarySearch(二分查找)

```C++
int searchInsert(vector<int>& nums, int target) {
    int n = nums.size();
    int left = 0, right = n - 1, ans = n;
    while (left <= right) {
        int mid = ((right - left) >> 1) + left;
        if (target <= nums[mid]) {
            ans = mid;
            right = mid - 1;
        } else {
            left = mid + 1;
        }
    }
    return ans;
}
```

## LowerBound

sort.Search Go 官方实现

```Go
func Search(n int, f func(int) bool) int {
	i, j := 0, n
	for i < j {
		h := int(uint(i+j) >> 1) // avoid overflow when computing h
		// i ≤ h < j
		if !f(h) {
			i = h + 1 // preserves f(i-1) == false
		} else {
			j = h // preserves f(j) == true
		}
	}
	return i
}
```

### "31. Next Permutation" Golang

```Go
func nextPermutation(nums []int)  {
    if len(nums) < 2 {
        return
    }

    l := len(nums) - 1
    for ; l > 0 && nums[l-1] >= nums[l]; l-- {
    }

    if l > 0 {
        r := l
        for i := l; i < len(nums) && nums[i] > nums[l-1]; r, i = i, i+1 {
        }
        nums[l-1], nums[r] = nums[r], nums[l-1]
    }

    for i, j := l, len(nums)-1; i < j; i, j = i+1, j-1 {
        nums[i], nums[j] = nums[j], nums[i]
    }
}
```

### "73. Set Matrix Zeroes"

```C++
void setZeroes(vector<vector<int>>& matrix) {
    if (matrix.empty() || matrix[0].empty()) return;
    const int M = matrix.size();
    const int N = matrix[0].size();
    bool isCol = false;
    for (int x = 0; x < M; ++x) {
        if (matrix[x][0] == 0)	// 记录未来1列是否置0
            isCol = true;
        for (int y = 1; y < N; ++y) {	// 从第2列遍历，把第1列作为标志位列
            if (matrix[x][y] == 0)
                matrix[0][y] = matrix[x][0] = 0;
        }
    }
    for (int x = M-1; x >= 0; --x) {	// 倒叙行遍历，利用第一行的标记功能
        for (int y = 1; y < N; ++y) {
            if (matrix[0][y] == 0 || matrix[x][0] == 0)
                matrix[x][y] = 0;
        }
        if (isCol) {
            matrix[x][0] = 0;
        }
    }
}
```

### "74. Search a 2D Matrix" LowerBound

```Go
func searchMatrix(matrix [][]int, target int) bool {
    m, n := len(matrix), len(matrix[0])
    i := sort.Search(m*n, func(i int) bool {return matrix[i/n][i%n] >= target})
    return i<m*n && matrix[i/n][i%n] == target
}
```

### "90. Subsets II" bit mask

```Go
func subsetsWithDup(nums []int) (ret [][]int) {
    sort.Ints(nums)
outer:
    for n, mask := len(nums), 0; mask < 1<<n; mask++ {
        t := make([]int, 0, bits.OnesCount(uint(mask)))
        for i, v := range nums {
            if (mask>>i)&1 > 0 {
                if i > 0 && (mask>>(i-1))&1 == 0 && v == nums[i-1] {
                    continue outer
                }
                t = append(t, v)
            }
        }
        ret = append(ret, t)
    }
    return ret
}
```

### "169. Majority Element" Golang

```Go
func majorityElement(nums []int) int {
    tar, cnt := 0, 0
    for _, n := range nums {
        if n == tar {
            cnt++
        } else if cnt > 0 {
            cnt--
        } else {
            tar = n
            cnt++
        }
    }

    if cnt > 0 {
        sum := 0
        for _, n := range nums {
            if n == tar {
                sum++
            }
        }
        if sum > len(nums)>>1 {
            return tar
        }
    }
    return -1
}
```

### "229. Majority Element II"

```Go
func majorityElement(nums []int) (ret []int) {
    a, b, aCnt, bCnt := 0, 0, 0, 0
    n := len(nums)
    for _, ele := range nums {
        if ele == a {
            aCnt++
            continue
        } else if ele == b {
            bCnt++
            continue
        }

        if aCnt == 0 {
            a = ele
            aCnt++
            continue
        } else if bCnt == 0 {
            b = ele
            bCnt++
            continue
        }

        aCnt--
        bCnt--
    }

    aCnt, bCnt = 0, 0
    for _, ele := range nums {
        if ele == a {
            aCnt++
        } else if ele == b {
            bCnt++
        }
    }

    if aCnt > n/3 {
        ret = append(ret, a)
    }
    if bCnt > n/3 {
        ret = append(ret, b)
    }

    return ret
}
```

### "179. Largest Number"

```Go
func largestNumber(nums []int) string {
    sort.Slice(nums, func(a, b int) bool {
        var factorA, factorB int64 = 10, 10
        for factorA <= nums[b] {
            factorA *= 10
        }
        for factorB <= nums[a] {
            factorB *= 10
        }
        return factorA*nums[a]+nums[b] > factorB*nums[b]+nums[a]
    })
    if nums[0] == 0 {
        return "0"
    }

    ret := []byte{}
    for _, v := range nums {
        ret = append(ret, strconv.Itoa(v)...)
    }
    return string(ret)
}
```

### "448. Find All Numbers Disappeared in an Array"

```Go
func findDisappearedNumbers(nums []int) (ret []int) {
    for i := 0; i < len(nums); i++ {
        cur := nums[i]
        if cur < 0 {
            cur *= -1
        }
        if nums[cur-1] > 0 {
            nums[cur-1] *= -1
        }
    }
    for i := 0; i < len(nums); i++ {
        if nums[i] > 0 {
            ret = append(ret, i+1)
        }
    }
    return ret
}
```

### "363. Max Sum of Rectangle No Larger Than K"

```Go
func maxSumSubmatrix(matrix [][]int, k int) (ret int) {
    M, N := len(matrix), len(matrix[0])
    dp := make([][]int, M+1)
    for i := 0; i <= M; i++ {
        dp[i] = make([]int, N+1)
    }
    for i := 1; i <= M; i++ {
        for j:=1; j <= N; j++ {
            dp[i][j] = dp[i-1][j]+dp[i][j-1]-dp[i-1][j-1]+matrix[i-1][j-1]
        }
    }
    ret = math.MinInt32
    for x1 := 0; x1 < M; x1++ {
        for y1 := 0; y1 < N; y1++ {
            for x2 := x1+1; x2 <= M; x2++ {
                for y2 := y1+1; y2 <= N; y2++ {
                    tmp := dp[x2][y2]-dp[x1][y2]-dp[x2][y1]+dp[x1][y1]
                    if tmp == k {
                        return k
                    } else if tmp <= k && tmp > ret {
                        ret = tmp
                    }
                }
            }
        }
    }
    return ret
}
```

### "581. Shortest Unsorted Continuous Subarray"

```Go
func findUnsortedSubarray(nums []int) int {
    n := len(nums)
    minVal, maxVal := math.MaxInt32, math.MinInt32
    l, r := -1, -1
    for i := 0; i < n; i++ {
        if maxVal <= nums[i] {
            maxVal = nums[i]
        } else {
            r = i
        }
        if minVal >= nums[n-i-1] {
            minVal = nums[n-i-1]
        } else {
            l = n-i-1
        }
    }
    if r == -1 {
        return 0
    }
    return r-l+1
}
```

### "918. Maximum Sum Circular Subarray"

```Go
func maxSubarraySumCircular(nums []int) int {
    n, s := len(nums), 0
    for _, num := range nums {
        s += num
    }
    return max(kadane(nums, 1), s+kadane(nums[:n-1], -1), s+kadane(nums[1:], -1))
}

func kadane(nums []int, sign int) (ret int) {
    ret = math.MinInt32
    sum := 0
    for _, num := range nums {
        sum = max(sum+num*sign, sign*num)
        ret = max(ret, sum)
    }
    return ret
}

func max(nums ...int) (ret int) {
    ret = math.MinInt32
    for _, num := range nums {
        if num > ret {
            ret = num
        }
    }
    return ret
}
```

### "1109. Corporate Flight Bookings"

```Go
func corpFlightBookings(bookings [][]int, n int) []int {
    diff := make([]int, n)
    for _, b := range bookings {
        diff[b[0]-1] += b[2]
        if b[1] < n {
            diff[b[1]] -= b[2]
        }
    }
    for i := 1; i < n; i++ {
        diff[i] += diff[i-1]
    }
    return diff
}
```

### "剑指 Offer 03. 数组中重复的数字 LCOF"

```Go
func findRepeatNumber(nums []int) int {
    for i := range nums {
        for nums[i] != i {
            if nums[i] == nums[nums[i]] {
                return nums[i]
            }
            nums[i], nums[nums[i]] = nums[nums[i]], nums[i]
        }
    }
    return 0
}
```
