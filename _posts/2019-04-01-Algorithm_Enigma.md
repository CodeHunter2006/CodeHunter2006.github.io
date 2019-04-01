---
layout: post
title:  "算法学习，Enigma密码生成器"
date:   2019-04-01 9:00:00 +0800
tags: Algorithm
---
### 题目
在网上看到一道有意思的算法题：在二战时，德军使用一个密码生成器Enigma对通信进行加密。<br/>
Enigma源自于希腊文，英语解释为“谜，不可思议的东西”，用几个机械转盘+电子线路组成，可以生成较强的密码，相比"凯撒法"的简单替换非常难破解，在二战初期帮助纳粹取得了优势。<br/>
题目就是给出三个参数（最小值、最大值、转盘数)写出可能的Enigma密码总数，要求第一个数与后边的数互质，由于结果数特别大需要对特定值取余。

<br/>
### 算法流程
* 首先计算因式分解需要的质数表
* 利用质数表对每个可用的数进行因式分解
* 利用容斥原理，对每两个数进行比较，记录每个数可能与它组合的数的数量
* 最后对所有可能性进行计算加总，注意加总时对特定值取余

<br/>
### 算法示例
```
/*
challenge: 
use n numbers combine a number, every number shuould bigger than minNum and lower than maxNum, first number should coprime with others. return the count of all possible numbers.

solve: 
factorize all valid numbers, check common factor of all numbers. 1 calculate to get prime list. 2 factorize every valid numbers. 3 check intersection of every number pair to count coprime number pairs. 4 get result acording to the n - 1 power of coprime count.

notice:
When calculate factors array of every number, 1 should be excluded, but the number should be included while the number is a prime itself, for common factor check process will check small prime too.
*/

void getPrimes(vector<int> &primes, int n) {
    vector<int> temp(n + 1, 1);
    temp[0] = temp[1] = 0;
    for (int i = 2; i <= n; ++i) {
        if (temp[i] == 0) continue;
        for (int j = 1; i + i * j <= n; ++j)
            temp[i + i * j] = 0;
    }
    for (int i = 0; i <= n; ++i) {
        if (temp[i] == 1)
            primes.push_back(i);
    }
}
void getFactors(vector<vector<int>> &factors, int minValue, int maxValue) {
    const int N = maxValue - minValue + 1;
    vector<int> primes;
    getPrimes(primes, maxValue);
    for (int i = minValue; i <= maxValue; ++i) {
        if (i == 1) continue;
        for (int cur = i, index = 0, pre = -1; cur != 1;) {
            if (cur % primes[index] == 0) {
                if (index != pre)
                    factors[i - minValue].push_back(primes[index]);
                cur /= primes[index];
                pre = index;
            } else {
                index++;
            }
        }
    }
}
bool hasCommon(vector<int> &v1, vector<int> &v2) {
    int i = 0, j = 0;
    while (i < v1.size() && j < v2.size()) {
        if (v1[i] == v2[j])
            return true;
        else if (v1[i] > v2[j])
            j++;
        else
            i++;
    }
    return false;
}
int calculateTotalRotorConfiguration(int rotorCount, int minRotorValue, int maxRotorValue) {
    const int N = maxRotorValue - minRotorValue + 1, MOD = 1e9 + 7;
    vector<vector<int>> factors(N);
    getFactors(factors, minRotorValue, maxRotorValue);
    vector<int> coprimeNums(N);
    for (int i = 0; i < N; ++i) {
        for (int j = i + 1; j < N; ++j) {
            if (!hasCommon(factors[i], factors[j])) {
                coprimeNums[i]++;
                coprimeNums[j]++;
            }
        }
    }
    int res = 0;
    for (auto i : coprimeNums) {
        res += pow(i, rotorCount - 1);
        res %= MOD;
    }
    return res;
}
```
