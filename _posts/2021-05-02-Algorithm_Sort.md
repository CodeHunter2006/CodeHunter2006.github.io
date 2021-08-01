---
layout: post
title: "Algorithm Sort"
date: 2021-05-02 09:00:00 +0800
tags: Algorithm Leetcode
---

记录 Sort 的算法实现

### "164. Maximum Gap"

```Go
// Radix sort
func maximumGap(nums []int) (ret int) {
    n := len(nums)
    if n < 2 {
        return 0
    }
    maxVal := max(nums...)
    buf := make([]int, n)
    for exp := 1; exp <= maxVal; exp *= 10 {
        count := make([]int, 10)
        for _, num := range nums {
            digit := num / exp % 10
            count[digit]++
        }
        for i := 1; i < len(count); i++ {
            count[i] += count[i-1]
        }
        for i := n-1; i >= 0; i-- {
            digit := nums[i] / exp % 10
            buf[count[digit]-1] = nums[i]
            count[digit]--
        }
        copy(nums, buf)
    }

    for i := 1; i < n; i++ {
        ret = max(ret, nums[i]-nums[i-1])
    }
    return ret
}
```

```Go
// Bucket sort
func maximumGap(nums []int) (ret int) {
    n := len(nums)
    if n < 2 {
        return 0
    }

    minVal, maxVal := min(nums...), max(nums...)
    d := max(1, (maxVal-minVal)/(n-1))  // min distance
    bucketSize := (maxVal-minVal)/d+1
    b := make([][2]int, bucketSize)
    for i := range b {
        b[i] = [2]int{-1,-1}    // min, max
    }
    for _, num := range nums {
        idx := (num-minVal)/d
        if b[idx][0] == -1 {
            b[idx] = [2]int{num, num}
        } else {
            b[idx][0] = min(b[idx][0], num)
            b[idx][1] = max(b[idx][1], num)
        }
    }

    for l,i := 0,1; i < len(b); i++ {
        if b[i][0] == -1 {
            continue
        }
        ret = max(ret, b[i][0]-b[l][1])
        l = i
    }
    return ret
}
```

### "215. Kth Largest Element in an Array"

```Go
func findKthLargest(nums []int, k int) int {
    sort.Sort(sort.Reverse(sort.IntSlice(nums)))
    return nums[k-1]
}
```

```Go
func findKthLargest(nums []int, k int) int {
    rand.Seed(time.Now().UnixNano())
    l, r, tar := 0, len(nums)-1, len(nums)-k
    for {
        index := partition(nums, l, r)
        if index == tar {
            return nums[index]
        } else if index > tar {
            r = index - 1
        } else {
            l = index + 1
        }
    }
    return -1
}

func partition(nums []int, l, r int) (ret int) {
    x := rand.Intn(r-l+1) + l   // 随机取 pivot 优化，nums[r] == pivot
    nums[x], nums[r] = nums[r], nums[x]
    for i := l; i < r; i++ {
        if nums[i] <= nums[r] {
           nums[l], nums[i] = nums[i], nums[l]
           l++
        }
    }
    nums[l], nums[r] = nums[r], nums[l]
    return l
}
```

### "274. H-Index"

```Go
// Count sort
func hIndex(citations []int) (h int) {
    n := len(citations)
    counter := make([]int, n+1)
    for _, citation := range citations {
        if citation >= n {
            counter[n]++
        } else {
            counter[citation]++
        }
    }
    for i, tot := n, 0; i >= 0; i-- {
        tot += counter[i]
        if tot >= i {
            return i
        }
    }
    return 0
}
```

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

### "645. Set Mismatch"

```Go
func findErrorNums(nums []int) (ret []int) {
    for i := 0; i < len(nums); i++ {
        for nums[i] != 0 && nums[i]-1 != i {
            if nums[nums[i]-1] == nums[i] {
                ret = append(ret, nums[i])
                nums[i] = 0
                break
            }
            nums[i], nums[nums[i]-1] = nums[nums[i]-1], nums[i]
        }
    }

    for i, v := range nums {
        if v == 0 {
            ret = append(ret, i+1)
            break
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

### "1833. Maximum Ice Cream Bars"

```Go
func maxIceCream(costs []int, coins int) (ans int) {
    const MX int = 1e5
    freq := [MX + 1]int{}
    for _, c := range costs {
        freq[c]++
    }
    for i := 1; i <= MX && coins >= i; i++ {
        c := min(freq[i], coins/i)
        ans += c
        coins -= i * c
    }
    return
}
```

### "1846. Maximum Element After Decreasing and Rearranging"

```Go
func maximumElementAfterDecrementingAndRearranging(arr []int) (maxVal int) {
    n := len(arr)
    bucket := make([]int, n+1)
    for _, a := range arr {
        bucket[min(a, n)]++
    }
    for i := 1; i <= n; i++ {
        maxVal = min(i, maxVal+bucket[i])
    }
    return maxVal
}
```
