---
layout: post
title: "Leetcode Binary Operation"
date: 2020-08-02 23:00:00 +0800
tags: Algorithm Leetcode
---

# 常用算法

## 原码、补码

在计算机中数字是以**补码**表示的，第一位是符号位(正数 0、负数 1)，后面以补码表示

正数：`补码 = 反码 = 原码`
负数：`补码 = 正数反码 + 1`

## XOR

- 利用异或运算交换变量

```Go
func xorSwap() {
    a, b := 1, 2
    a ^= b  // 将"1^2"的结果保存到a
    b ^= a  // 利用"2"将"1^2"中的"1"取出，保存到b
    a ^= b  // 利用"1"将"1^2"中的"2"取出，保存到a
}

func mathSwap() {
    a, b := 1, 2
    a = a + b
    b = a - b
    a = a - b
}
```

- 如示例，`异或`运算同时具有算数`+-`能力，可以利用其做交换操作
- `^`异或运算，可以将两个元素的"混合"状态记录下来，相当于算数中的"求和"
- 异或具有和`+-`相同的交换性`a^b == b^a`、结合性`x = a^b; x^c == a^b^c`
- 异或具有特殊的"自反"性，相当于`+`之后执行`-`，`a^b^b == a`
- 异或有特性：`c = a^b => a = c^b`

## lowbit 找到最右边的 1

负数的补码为了避`+0`和`-0`重合，整体数值`+1`，这样对于同一数值，正、负数的补码求 & 恰好可以获得最后一个 1。
如：5(b0...0101) 则 -5 为`负数补码 = 正数原码的反码 + 1 = b1...1010 + b1 = b1...1011`。
`5(b0...0101) & -5(b1...1011) = b0...0001`只保留了最右边一位 1

```Go
x & (-x)
```

## 去掉二进制末尾的 1

用下面方法可以将任意整型去掉末尾的 1，比如`0b1010 -> 0b1000`

```Go
ret := x & (x-1)

// 也可以 x - lowbit(x)
ret := x - (x & (-x))
```

## OnesCount

计算一个二进制数中位置为 1 的数量。
可以用遍历每个位的方法，但是需要对每个位进行判断。但是更简单的是利用`x &= x - 1`的方式去掉最后一个 1，然后迭代取得结果。
一般语言都会内置该函数，比如 Go 中`bits.OnesCount`

```Go
func onesCount(x int) (ones int) {
    for ; x > 0; x &= x - 1 {
        ones++
    }
    return
}
```

## 枚举二进制子集

用下面算法，可以输出某个二进制数的全部子集，所谓子集，就是把部分原本 1 的位置为 0。
比如`101`的子集是：`101`、`100`、`001`、`0`

- 子集包含原二进制数和 0

```Go
func getSubset(input uint32) (ret []uint32) {
    subset := input
    for {
        ret = append(ret, subset)
        subset = (subset-1)&input
        if subset == input {
            break
        }
    }
    return ret
}
```

## 数字电路设计

以"137. Single Number II"为例，解释如何从按位状态机推导出最终的简化表达式。

- 数字电路设计基础：
  - 了解简单的门电路（例如与门、异或门等）
  - 给定数字电路输入和输出（真值表），使用门电路设计出一种满足要求的数字电路结构

### 门电路表示

- 我们将用到 4 中门电路：
  - 非门：我们用`A'`表示输入为`A`的非门的输出；
  - 与门：我们用`AB`表示输入为`A`和`B`的与门的输出。
    由于「与运算」具有结合律，因此如果同时用了多个与门（例如将`A`和`B`进行与运算后，再和`C`进行与运算），
    我们可以将多个输入写在一起（例如`ABC`）；
  - 或门：我们用`A+B`表示输入为`A`和`B`的或门的输出。
    同样地，多个或门可以写在一起（例如`A+B+C`）；
  - 异或门：我们用`A⊕B`表示输入为`A`和`B`的异或门的输出。
    同样的，多个异或门可以写在一起（例如`A⊕B⊕C`）。

### 初步思路

假设我们已经实现了遍历 32 位进行按位状态机的算法，我们考虑用二进制运算同时处理 32 位。
我们的目标是求累加后 3 的余数(0/1/2)，由于二进制只能存储 0/1，所以我们想存储 2 就要利用两个二进制位。
假设$x_i$表示第 i 个位，由两个二进制位表示，那么 x 增 1 时可能有三种循环迁移状态：`00`->`01`->`11`->`00`->...
我们设 a 为高位、b 为低位，那么根据上面的状态可以画出真值表：

| ${a_i}{b_i}$ | $x_i$ | $新{a_i}{b_i}$ |
| ------------ | ----- | -------------- |
| 00           | 0     | 00             |
| 00           | 1     | 01             |
| 01           | 0     | 01             |
| 01           | 1     | 10             |
| 10           | 0     | 10             |
| 10           | 1     | 00             |

考虑输出$a_i$时，真值表：

| ${a_i}{b_i}$ | $x_i$ | $新{a_i}$ |
| ------------ | ----- | --------- |
| 00           | 0     | 0         |
| 00           | 1     | 0         |
| 01           | 0     | 0         |
| 01           | 1     | 1         |
| 10           | 0     | 1         |
| 10           | 1     | 0         |

可设计出电路：

$a_i={a_i'b_ix_i}+{a_ib_i'x_i'}$

考虑输出$b_i$时，真值表：

| ${a_i}{b_i}$ | $x_i$ | $新{b_i}$ |
| ------------ | ----- | --------- |
| 00           | 0     | 0         |
| 00           | 1     | 1         |
| 01           | 0     | 1         |
| 01           | 1     | 0         |
| 10           | 0     | 0         |
| 10           | 1     | 0         |

可设计出电路：

$b_i={a_i'b_i'x_i}+{a_i'b_ix_i'}=a_i'(b_i⊕x_i)$

将上面的电路逻辑转换为等价的整数位运算为：

`a = (~a & b & x) | (a & ~b & ~x)`
`b = ~a & (b^x)`

其中`~ & | ^`分别表示按位"非"、"与"、"或"、"异或"运算。(Golang 中单目运算符`^`表示"非")

得出的 Go 关键代码：`a, b = b&^a&num|a&^b&^num, (b^num)&^a`

### 数字电路设计优化

由于上面的 a 规则较为麻烦，可以考虑将上面"同时计算"改为"分别计算"，即先计算"新 b"再拿"新 b"计算 a。
将上面求 a 的真值表用"$新b_i$"替换"$旧b_i$"可得到真值表：

| ${a_i},{新b_i}$ | $x_i$ | $新{a_i}$ |
| --------------- | ----- | --------- |
| 00              | 0     | 0         |
| 01              | 1     | 0         |
| 01              | 0     | 0         |
| 00              | 1     | 1         |
| 10              | 0     | 1         |
| 10              | 1     | 0         |

可设计出电路：

$a_i={a_i'b_i'x_i}+{a_ib_i'x_i'}=b_i'(a_i⊕x_i)$

最终转换规则：(要保证顺序，先计算新 b)

`b = ~a & (b^x)`
`a = ~b & (a^x)`

# 题目

### 剑指 Offer 56 - I. 数组中数字出现的次数 / 136. Single Number

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

### 剑指 Offer 56 - II. 数组中数字出现的次数 II / 137. Single Number II

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

- 改进算法：(有限状态自动机)
  1. 按照上面的位变化可以看出，每个位实际上是在三个数字间变化：0->1->2->0...
  2. 每当新的值是 1 时，原有状态就跳转到下一状态；而新的值是 0 时保持原状态不变化
  3. 由于二进制只有两个状态 0 1，无法表示上面的三个状态，所以考虑用两个二进制位来表示三个状态
  4. 其中一个叫低位 low，一个叫高位 high，那么上面三个状态变化表示为：00->01->10->00...
  5. 要计算 low 的当前值，需要结合新值 num、高位值 high，按照 & ^ 的逻辑，可表示为公式`low = low ^ num & ~high`
  6. 经分析，计算 high 的公式和 low 的公式一致`high = high ^ num & ~low`
  7. 因为我们要对所有位同时独立计算，所以用 32 位整型 lows 和 highs 表示多个位
  8. 因为答案只有一个数，对应的位状态只会是 1，所以最后只需要返回 lows 就可以了

```Go
func singleNumber(nums []int) int {
    lows, highs := 0, 0
    for _, n := range nums {
        lows = lows ^ n & ^highs
        highs = highs ^ n & ^lows
    }
    return lows
}
```

### "421. Maximum XOR of Two Numbers in an Array"

```Go
func findMaximumXOR(nums []int) (x int) {
    const highBitIdx = 30
    for k := highBitIdx; k >= 0; k-- {
        preBit := make(map[int]bool, len(nums))
        for _, v := range nums {
            preBit[v>>k] = true
        }

        preKBitX, found := x | 1<<k, false
        for _, v := range nums {
            if preBit[v>>k ^ preKBitX>>k] {
                found = true
                break
            }
        }

        if found {
            x = preKBitX
        }
    }
    return x
}
```

### "461. Hamming Distance" Golang

```Golang
func hammingDistance(x int, y int) (ret int) {
    for x = x^y; x > 0; x = x & (x-1) {
        ret++
    }
    return ret
}
```

### "477. Total Hamming Distance"

```Go
func totalHammingDistance(nums []int) (ret int) {
    for i := 0; i < 30; i++ {
        sum := 0
        for _, n := range nums {
            sum += n>>i & 1
        }
        ret += sum * (len(nums)-sum)
    }
    return ret
}
```

### "1178. Number of Valid Words for Each Puzzle"

```Golang
func findNumOfValidWords(words []string, puzzles []string) []int {
    const PUZZLE_LENGTH = 7
    cntMap := make(map[uint32]int)
    for _, w := range words {
        var bitmap uint32
        for _, c := range w {
            bitmap |= 1 << (c-'a')
        }
        if bits.OnesCount32(bitmap) <= PUZZLE_LENGTH {
            cntMap[bitmap]++
        }
    }

    ret := make([]int, len(puzzles))
    for i, p := range puzzles {
        var first uint32 = 1 << (p[0]-'a')
        var bitmap uint32
        for _, c := range p[1:] {
            bitmap |= 1 << (c-'a')
        }

        subSet := bitmap
        for {
            ret[i] += cntMap[subSet|first]
            subSet = (subSet-1)&bitmap
            if subSet == bitmap {
                break
            }
        }
    }
    return ret
}
```

### "1310. XOR Queries of a Subarray"

```Go
func xorQueries(arr []int, queries [][]int) (ret []int) {
    preSum := make([]int, len(arr)+1)
    for i, v := range arr {
        preSum[i+1] = preSum[i] ^ v
    }
    ret = make([]int, len(queries))
    for i, q := range queries {
        ret[i] = preSum[q[1]+1] ^ preSum[q[0]]
    }
    return ret
}
```

### "1442. Count Triplets That Can Form Two Arrays of Equal XOR"

```Go
func countTriplets(arr []int) (ans int) {
    cnt, total := map[int]int{}, map[int]int{}
    for k, s := 0, 0; k < len(arr); k++ {
        if m, has := cnt[s^arr[k]]; has {
            ans += m*k - total[s^arr[k]]
        }
        cnt[s]++
        total[s] += k
        s ^= arr[k]
    }
    return
}
```

### "1720. Decode XORed Array"

```Go
func decode(encoded []int, first int) []int {
    ret := make([]int, len(encoded)+1)
    ret[0] = first
    for i := 0; i < len(encoded); i++ {
        ret[i+1] = ret[i] ^ encoded[i]
    }
    return ret
}
```

### "1734. Decode XORed Permutation"

```Go
func decode(encoded []int) []int {
    N := len(encoded)+1
    all, other := 0, 0
    for i := 1; i <= N; i++ {
        all ^= i
    }
    for i, v := range encoded {
        if i & 1 == 1 {
            other ^= v
        }
    }

    ret := make([]int, N)
    ret[0] = all ^ other // first
    for i := 1; i < N; i++ {
        ret[i] = ret[i-1] ^ encoded[i-1]
    }
    return ret
}
```

### "1738. Find Kth Largest XOR Coordinate Value"

```Go
func quickSelect(a []int, k int) int {
    rand.Shuffle(len(a), func(i, j int) { a[i], a[j] = a[j], a[i] })
    for l, r := 0, len(a)-1; l < r; {
        v := a[l]
        i, j := l, r+1
        for {
            for i++; i < r && a[i] < v; i++ {
            }
            for j--; j > l && a[j] > v; j-- {
            }
            if i >= j {
                break
            }
            a[i], a[j] = a[j], a[i]
        }
        a[l], a[j] = a[j], v
        if j == k {
            break
        } else if j < k {
            l = j + 1
        } else {
            r = j - 1
        }
    }
    return a[k]
}

func kthLargestValue(matrix [][]int, k int) int {
    m, n := len(matrix), len(matrix[0])
    results := make([]int, 0, m*n)
    pre := make([][]int, m+1)
    pre[0] = make([]int, n+1)
    for i, row := range matrix {
        pre[i+1] = make([]int, n+1)
        for j, val := range row {
            pre[i+1][j+1] = pre[i+1][j] ^ pre[i][j+1] ^ pre[i][j] ^ val
            results = append(results, pre[i+1][j+1])
        }
    }
    return quickSelect(results, m*n-k)
}

```
