---
layout: post
title: "Leetcode Binary Operation"
date: 2020-08-02 23:00:00 +0800
tags: Algorithm Leetcode
---

## 剑指 Offer 56 - I. 数组中数字出现的次数

[题目](https://leetcode-cn.com/problems/shu-zu-zhong-shu-zi-chu-xian-de-ci-shu-lcof/)、官方解答[异或运算](https://leetcode-cn.com/problems/shu-zu-zhong-shu-zi-chu-xian-de-ci-shu-lcof/solution/shu-zu-zhong-shu-zi-chu-xian-de-ci-shu-by-leetcode/)

- 直观思路
  这道题要求`时间复杂度O(n)，空间复杂度O(1)`，用直观思路不可能实现。需要利用二进制异或运算。
- 解体思路第一步
  1. 一位二进制，对自己做异或运算将得到 0。这是异或运算的基本定义：1^1 = 0^0 = 0; 1^0 = 0^1 = 1;
  2. 一个整型由多个二进制位组成，这个整型由这些二进制位的组合唯一确定。这个整型对自己进行异或时，每一位将进行异或，最终结果为 0。
  3. 一个数组，元素数量为偶数，并且所有元素在数组中都出现两次，那么数组内元素全部遍历异或一遍，将得到 0，因为每一位二进制信息必然与重复的自己相"抵消"。
  4. 在上面数组中加入一个没出现过的元素，数组元素数量为奇数，那么同样组内元素异或一遍，最终得到的就是这个独立的新元素，因为其他元素已经与自己相"抵消"。
- 解体思路第二步
  1. 这道题是有两个独立数存在的数组，如果能够把这个数组拆分一下，拆成两个数组，并且每个数组分别包含一个独立数，就可以用上面数组内异或的方法找到独立数。
  2. 可以对这两个数取一次异或运算，得到的整型以二进制表达，某一位一定是 1，这一位原来两个独立数一定不相同，可以利用这一位进行分组。
  3. 对初始数组全部元素异或，就可以取得上面的数，因为其他重复元素将自我抵消，不影响结果。
  4. 按照找到的二进制位进行分组，0/1 各分到一组，然后分别执行组内异或，即可找到最终的结果——两个独立数。

```Go
func singleNumbers(nums []int) []int {
    separator := 0
    for _, n := range nums {
        separator ^= n
    }

    // 查找第一个位置差异点
    position := 0
    for ; position < int(unsafe.Sizeof(nums[0])); position++ {
        if ((1 << position) & separator) > 0 {
            break
        }
    }

    res := []int{0, 0}
    for _, n := range nums {
        if (n & (1 << position)) > 0 {
            res[0] ^= n
        } else {
            res[1] ^= n
        }
    }

    return res
}
```

- 改进点：
  1. 利用`lowbit(x)`优化
     `lowbit(x)`函数的功能是返回某个二进制数最右边的一个 1 (可以算是"特征位"吧)所在的位置对应的数，公式为`lowbit(x) = x & -x = x & (-x+1)`
     自己简单用二进制例子推导一下(如 0010 & 1110 = 0010)，会发现这就是补码的特性，可以代替上面查找第一个 1 所在位置的算法。
  2. 在特定位置判断时，可以直接使用`!= 0`进行比较，效率高一点点
  3. 在运算过程中用变量替代 slice，效率又能高一点点

```Go
func singleNumbers(nums []int) []int {
    a, b, x := 0, 0, 0

    for _, num := range nums {
        x ^= num
    }
    x = x & -x  // 求出 lowbit

    for _, num := range nums {
        if num & x != 0 {
            a ^= num
        } else {
            b ^= num
        }
    }

    return []int{a, b}
}
```

## 剑指 Offer 56 - II. 数组中数字出现的次数 II

[题目](https://leetcode-cn.com/problems/shu-zu-zhong-shu-zi-chu-xian-de-ci-shu-ii-lcof/)、[官方解答](https://leetcode-cn.com/problems/shu-zu-zhong-shu-zi-chu-xian-de-ci-shu-ii-lcof/solution/mian-shi-ti-56-ii-shu-zu-zhong-shu-zi-chu-xian-d-4/)

- 直观思路
  利用 map 统计数字出现的次数，将出现三次的都剔除掉
- 改进思路
  本题如果限定和上题一样的条件`时间复杂度O(n)，空间复杂度O(1)`，那就不能用 map。可以利用每个 bit 会出现三次的特点，对所有位置的 bit 进行统计。由于 bit 数量是固定的，所以空间复杂度是`O(1)`

```Go
func singleNumber(nums []int) int {
    const SIZE = 32
    bitSli := make([]int, SIZE, SIZE)

    for _, n := range nums {
        for i := 0; i < SIZE; i++ {
            if (n & (1 << i)) != 0 {
                bitSli[i]++
            }
        }
    }

    res := 0
    for i := range bitSli {
        if bitSli[i] % 3 != 0 {
            res |= 1 << i
        }
    }

    return res
}
```
