---
layout: post
title: "Algorithm Two Pointers"
date: 2020-12-27 09:00:00 +0800
tags: Algorithm Leetcode
---

记录 Two Pointers 的算法实现

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
