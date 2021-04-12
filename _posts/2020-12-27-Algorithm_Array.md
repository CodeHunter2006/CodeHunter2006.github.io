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
    cur, count := 0, 0

    for _, n := range nums {
        if n == cur {
            count++
        } else if count > 1 {
            count--
        } else {
            cur = n
            count = 1
        }
    }

    return cur
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

### "912. Sort an Array" QuickSort C++

```C++
// 三数取中法优化函数，效果较好
template<typename T1>
    void selectPivotOfThree(vector<T1> &nums, int lo, int hi) {
        int m = (lo + hi) >> 1;
        if (nums[m] > nums[hi])
            swap(nums[m], nums[hi]);
        if (nums[lo] > nums[hi])
            swap(nums[lo], nums[hi]);
        if (nums[m] > nums[lo])
            swap(nums[lo], nums[m]);
    }

    // 随机取数法，
    template<typename T1>
    void selectPivotByRand(vector<T1> &nums, int lo, int hi) {
        int idx = lo + (rand() % (hi - lo + 1));
        swap(nums[lo], nums[idx]);
    }

    template<typename T1>
    void quickSort(vector<T1> &nums, int lo, int hi) {
        if (lo >= hi) return;  // 注意这里要及时退出
        //selectPivotOfThree<T1>(nums, lo, hi);
        selectPivotByRand(nums, lo, hi);
        T1 pivot = nums[lo];
        int l = lo, r = hi;
        while (l < r) {
            while (l < r && nums[r] >= pivot)
                r--;
            swap(nums[l], nums[r]);
            while (l < r && nums[l] <= pivot)
                l++;
            swap(nums[l], nums[r]);
        }
        quickSort(nums, lo, l);   // 注意这里不能是l-1
        quickSort(nums, l + 1, hi);
    }
	vector<int> sortArray(vector<int>& nums) {
		quickSort<int>(nums, 0, nums.size() - 1);
		return nums;
	}
```

### "912. Sort an Array" QuickSort Golang

```Golang
func quick(nums []int) {
    if len(nums) <= 1 {
        return
    }
    pivot, l, r := nums[0], 0, len(nums)-1
    for l < r {
        for l < r && nums[r] >= pivot {
            r--
        }
        nums[l], nums[r] = nums[r], nums[l]
        for l < r && nums[l] <= pivot {
            l++
        }
        nums[l], nums[r] = nums[r], nums[l]
    }
    quick(nums[:l])
    quick(nums[l+1:])
}
```
