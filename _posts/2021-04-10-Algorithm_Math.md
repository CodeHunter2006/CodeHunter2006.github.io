---
layout: post
title: "Algorithm Math"
date: 2021-04-10 22:00:00 +0800
tags: Algorithm Leetcode
---

记录 Math 的算法实现

# 基础算法

## 等差数列前 n 项求和公式

`Sn = n(a1+an)/2`

示例：
["413. Arithmetic Slices" Math]()

## 等比数列前 n 项求和公式

`Sn = a1(1 - q^n) / (1 - q)`

## GCD(Greatest Common Divisor)

**最大公约数**是指多个整数共有的约数中最大的一个。两个相同整数的最大公约数是这个整数自己。

最大公约数是经常用到的算法，可以用"辗转取余法"实现：

```Go
func gcd(a, b int) int {
    for a > 0 {
        a, b = b%a, a
    }
    return b
}
```

- 思路：
  1. 假设 a 是较小的数。其实 a b 无论谁是较小的数不影响结果
  2. 取余时可以找到本轮的公约数，直到 a==0 则 b 就是最后结果

## 同余定理

**同余定理**如果两数之差可以被 m 整除，那么两数分别对 m 取余的值相同。
`(a-b)%m == 0 <=> a%m == b%m == k`

经常结合 HashTable 记录 k 值，实现求和后的余数计算。

示例：
["974. Subarray Sums Divisible by K" HashTable Golang]()

# 题目实现

### "7. Reverse Integer"

```Go
func reverse(x int) (ret int) {
    for x != 0 {
        if ret < math.MinInt32/10 || ret > math.MaxInt32/10 {
            return 0
        }
        single := x%10
        x /= 10
        ret = ret*10 + single
    }
    return ret
}
```

### "65. Valid Number"

```C++
bool isNumber(string s) {
    int state = 0;
    int i = 0, j = s.size() - 1;    // trim
    while (i < s.size() && s[i] == ' ') i++;
    while (j >= 0 && s[j] == ' ') j--;
    for (; i <= j; ++i) {
        if (s[i] == '+' || s[i] == '-') {
            if (state == 0)
                state = 1;
            else if (state == 4)
                state = 6;
            else
                return false;
        } else if (isdigit(s[i])) {
            if (state == 0 || state == 1 || state == 2)
                state = 2;
            else if (state == 3)
                state = 3;
            else if (state == 4 || state == 5 || state == 6)
                state = 5;
            else if (state == 7 || state == 8)
                state = 8;
            else
                return false;
        } else if (s[i] == '.') {
            if (state == 0 || state == 1)
                state = 7;
            else if (state == 2)
                state = 3;
            else
                return false;
        } else if (s[i] == 'e') {
            if (state == 2 || state == 3 || state == 8)
                state = 4;
            else
                return false;
        } else {
            return false;
        }
    }
    return state == 2 || state == 3 || state == 5 || state == 8;
}
```

### "69. Sqrt(x)" Newton iteration algorithm

```C++
long r = x;
while (r*r > x)
    r = (r + x/r) / 2;
return r;
```

### "204. Count Primes" the Sieve of Eratosthenes

```C++
int countPrimes(int n) {
    if(n < 2) return 0;

    int count = 0;
    vector<bool> primes(n, true);
    primes[0] = primes[1] = false;

    for (int i = 0; i < n; ++i) {
        if (primes[i]) {
            for (int j = i*2; j < n; j += i)
                primes[j] = false;
        }
    }

    count = std::count(primes.begin(), primes.end(), true);
    return count;
}
```

### "233. Number of Digit One"

```C++
int ones = 0;
for (long m = 1; m <= n; m *= 10)
    ones += (n/m + 8) / 10 * m + (n/m % 10 == 1) * (n%m + 1);
return ones;
```

### "264. Ugly Number II" Math+DP

```Go
func nthUglyNumber(n int) int {
    dp := make([]int, n+1)
    dp[1] = 1
    p2, p3, p5 := 1, 1, 1
    for i := 2; i <= n; i++ {
        next2, next3, next5 := dp[p2]*2, dp[p3]*3, dp[p5]*5
        dp[i] = min(min(next2, next3), next5)
        if dp[i] == next2 {
            p2++
        }
        if dp[i] == next3 {
            p3++
        }
        if dp[i] == next5 {
            p5++
        }
    }
    return dp[n]
}
```

### "279. Perfect Squares" Lagrange's four-square theorem

```C++
int numSquares(int n) {
    int sq = sqrt(n);
    if (sq * sq == n)
        return 1;
    for (int i = 0; i <= sq; i++) {
        int t = n - i * i;
        for (int l = i, r = sq; l <= r;) {
            int df = l * l + r * r - t;
            if (!df)
                return i ? 3 : 2;
            df < 0 ? l++ : r--;
        }
    }
    return 4;
}
```

### "296. Best Meeting Point"

```C++
int minTotalDistance(vector<vector<int>>& grid) {
    if (grid.empty() || grid[0].empty()) return 0;
    const int M = grid.size();
    const int N = grid[0].size();
    vector<int> rows, cols;
    for (int x = 0; x < M; ++x) {
        for (int y = 0; y < N; ++y) {
            if (grid[x][y])
                rows.push_back(x);
        }
    }
    for (int y = 0; y < N; ++y) {
        for (int x = 0; x < M; ++x) {
            if (grid[x][y])
                cols.push_back(y);
        }
    }
    return getDistance(rows) + getDistance(cols);
}
int getDistance(vector<int> positions) {
    int res = 0, l = 0, r = positions.size() - 1;
    while (l < r)
        res += positions[r--] - positions[l++];
    return res;
}
```

### "313. Super Ugly Number"

```Go
func nthSuperUglyNumber(n int, primes []int) (ugly int) {
    m := map[int]bool{1:true}
    h := &myHeap{[]int{1}}
    for ; n > 0; n-- {
        ugly = heap.Pop(h).(int)
        for _, prime := range primes {
            if next := ugly*prime; !m[next] {
                m[next] = true
                heap.Push(h, next)
            }
        }
    }
    return ugly
}

type myHeap struct {
    sort.IntSlice
}
func (p *myHeap) Push(x interface{}) { p.IntSlice = append(p.IntSlice, x.(int)) }
func (p *myHeap) Pop() interface{} {
    l := len(p.IntSlice)
    tmp := p.IntSlice[l-1]
    p.IntSlice = p.IntSlice[:l-1]
    return tmp
}
```

### "365. Water and Jug Problem" "欧几里德算法/辗转相除法"

```C++
bool canMeasureWater(int x, int y, int z) {
    return z == 0 || z <= (long)x + y && z % gcd(x, y) == 0;
}
int gcd(int x, int y) {
    return y == 0 ? x : gcd(y, x % y);
}
```

### "413. Arithmetic Slices"

```Go
func numberOfArithmeticSlices(nums []int) (ret int) {
    diff, count := 2001, 0
    for i := 1; i < len(nums); i++ {
        if nums[i]-nums[i-1] == diff {
            count++
            ret += count
        } else {
            diff, count = nums[i]-nums[i-1], 0
        }
    }
    return ret
}
```

### "458. Poor Pigs"

```C++
int poorPigs(int buckets, int minutesToDie, int minutesToTest) {
    int states = minutesToTest / minutesToDie + 1;
    return ceil(log(buckets)/log(states));
}
```

### "523. Continuous Subarray Sum"

```Go
func checkSubarraySum(nums []int, k int) bool {
    m,sum := make(map[int]int),0
    for i,x := range nums {
        sum += x
        mod := sum%k
        if mod == 0 && i != 0 {
            return true
        }
        if tmp,ok := m[mod]; ok {
            if tmp+1 != i {
                return true
            }
        } else {
            m[mod] = i
        }
    }
    return false
}
```

### "633. Sum of Square Numbers"

```Go
func judgeSquareSum(c int) bool {
    l, r := 0, int(math.Sqrt(float64(c)))
    for l <= r {
        tmp := l*l + r*r
        if tmp == c {
            return true
        } else if tmp > c {
            r--
        } else {
            l++
        }
    }
    return false
}
```

### "753. Cracking the Safe" 花花酱 Hamiltonian

```C++
string crackSafe(int n, int k) {
    string res(n, '0');
    const int total_size = pow(k, n);
    unordered_set<string> visited;
    visited.insert(res);
    if (dfs(res, total_size, n, k, visited))
        return res;
    return "";
}
bool dfs(string &res, const int total_size, const int n, const int k, unordered_set<string> &visited) {
    if (visited.size() == total_size) return true;
    string prefix = res.substr(res.size() - n + 1, n - 1);
    for (char c = '0'; c < '0' + k; ++c) {
        prefix.push_back(c);
        if (!visited.count(prefix)) {
            res.push_back(c);
            visited.insert(prefix);
            if (dfs(res, total_size, n, k, visited))
                return true;
            visited.erase(prefix);
            res.pop_back();
        }
        prefix.pop_back();
    }
    return false;
}
```

### "780. Reaching Points"

```C++
bool reachingPoints(int sx, int sy, int tx, int ty) {
    while (sx < tx && sy < ty)
        if (tx < ty) ty %= tx;
    	else tx %= ty;
    return sx == tx && sy <= ty && (ty - sy) % sx == 0 ||
        sy == ty && sx <= tx && (tx - sx) % sy == 0;
}
```

### "810. Chalkboard XOR Game"

```Go
func xorGame(nums []int) bool {
    var sum int
    for i := range nums {
        sum ^= nums[i]
    }
    return sum == 0 || len(nums) % 2 == 0
}
```

### "829. Consecutive Numbers Sum"

```C++
int consecutiveNumbersSum(int N) {
    int count = 0;
    for (int k = 1; k * (k + 1) / 2 <= N; ++k) { // 设定了公式范围，对k循环
        if (2 * N % k == 0) { // 保证可以整除
            int y = 2 * N / k - k - 1;
            if (y % 2 == 0 && y >= 0) // 同样保证整除
                count++;
        }
    }
    return count;
}
```
