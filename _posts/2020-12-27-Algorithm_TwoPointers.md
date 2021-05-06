---
layout: post
title: "Algorithm Two Pointers"
date: 2020-12-27 09:00:00 +0800
tags: Algorithm Leetcode
---

记录 Two Pointers 的算法实现

### "实现 memcpy" C++

```C++
// 内存拷贝函数，源的内容允许被覆盖，但是目标的内容要完整
void* myMemcpy(void *dest, const void* src, unsigned int count) {
	if (dest == nullptr || src == nullptr || dest == src) return dest;

	char *pDest = (char *)dest;
	char *pSrc = (char *)src;
	if (pDest > pSrc && dest < (pSrc + count)) {
		pDest += count - 1;
		pSrc += count - 1;
		while (count--) {
			*pDest-- = *pSrc--;
		}
	} else {
		while (count--) {
			*pDest++ = *pSrc++;
		}
	}
	return dest;
}
```

### "11. Container With Most Water"

```Go
func min(a, b int) int {
    if a <= b {
        return a
    } else {
        return b
    }
}

func max(a, b int) int {
    if a >= b {
        return a
    } else {
        return b
    }
}

func maxArea(height []int) int {
    res, l, r := 0, 0, len(height) - 1
    for l < r {
        low := min(height[l], height[r])
        res = max(res, (r - l) * low)
        if height[l] == low {
            for l < r && height[l] <= low {
                l++
            }
        } else {
            for l < r && height[r] <= low {
                r--
            }
        }
    }

    return res
}
```

### "42. Trapping Rain Water" Golang

```Go
func max(a, b int) int {
    if a > b {
        return a
    }
    return b
}

func trap(height []int) (ret int) {
    sum := 0
    for lMax, rMax, l, r := 0, 0, 0, len(height)-1; l < r; {
        lMax = max(lMax, height[l])
        rMax = max(rMax, height[r])
        if height[l] < height[r] {
            sum += lMax - height[l]
            l++
        } else {
            sum += rMax - height[r]
            r--
        }
    }
    return sum
}
```

### "457. Circular Array Loop"

```C++
int getIndex(const int &i, const vector<int>& nums) {
    int n = nums.size();
    int step = i + nums[i];
    while(step < 0)
        step += n;
    return step%n;
}
bool circularArrayLoop(vector<int>& nums) {
    for ( int i=0 ; i<nums.size() ; i++ ) {
        // slow, fast index pointer to know there is a circle or not.
        int slow = i, fast = i;
        // "nums[slow] * nums[fast] > 0" to keep same direction and skip checked indices
        while ( nums[slow] * nums[fast] > 0 && nums[fast] * nums[getIndex(fast, nums)] > 0 ) {
            slow = getIndex(slow, nums);
            fast = getIndex(getIndex(fast, nums), nums);
            if ( slow == fast ) {
                // if circle length == 1
                if ( slow == getIndex(slow, nums) )
                    break;
                return true;
            }
        }

        slow = i;
        fast = getIndex(i, nums);
        // mark checked indices
        while ( nums[slow] * nums[fast] > 0 ) {
            nums[slow] = 0;
            slow = fast;
            fast = getIndex(slow, nums);
        }
    }
    return false;
}
```
