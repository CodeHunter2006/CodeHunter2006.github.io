---
layout: post
title: "Algorithm Divide and Conquer"
date: 2021-05-02 09:00:00 +0800
tags: Algorithm Leetcode
---

记录 Divide and Conquer 的算法实现

### "468. Validate IP Address"

```C++
const string validIPv6Chars = "0123456789abcdefABCDEF"; // consider upper or lowwer
bool isValidIPv4Block(string& block) {
    int num = 0;
    if (block.size() > 0 && block.size() <= 3) {
        for (int i = 0; i < block.size(); i++) {
            char c = block[i];
            // special case: if c is a leading zero and there are characters left
            if (!isalnum(c) || (i == 0 && c == '0' && block.size() > 1))
                return false;
            else {
                num *= 10;
                num += c - '0';
            }
        }
        return num <= 255;
    }
    return false;
}
bool isValidIPv6Block(string& block) {
    if (block.size() > 0 && block.size() <= 4) {
        for (int i = 0; i < block.size(); i++) {
            char c = block[i];
            if (validIPv6Chars.find(c) == string::npos)
                return false;
        }
        return true;
    }
    return false;
}
string validIPAddress(string IP) {
    string ans[3] = {"IPv4", "IPv6", "Neither"};
    stringstream ss(IP);
    string block;
    // ipv4 candidate
    if (IP.substr(0, 4).find('.') != string::npos) {
        for (int i = 0; i < 4; i++) {
            if (!getline(ss, block, '.') || !isValidIPv4Block(block))
                return ans[2];
        }
        return ss.eof() ? ans[0] : ans[2];
    }else if (IP.substr(0, 5).find(':') != string::npos) { // ipv6 candidate
        for (int i = 0; i < 8; i++) {
            if (!getline(ss, block, ':') || !isValidIPv6Block(block))
                return ans[2];
        }
        return ss.eof() ? ans[1] : ans[2];
    }
    return ans[2];
}
```

### "241. Different Ways to Add Parentheses"

```Go
func diffWaysToCompute(input string) []int {
    operators := []rune{}
    nums := []int{}
    inNum := false  // is a digital character
    for _, r := range []rune(input) {
        if r == '-' || r == '+' || r == '*' {
            inNum = false
            operators = append(operators, r)
        } else if inNum {
            nums[len(nums)-1] *= 10
            nums[len(nums)-1] += int(r - '0')
        } else {
            inNum = true
            nums = append(nums, int(r - '0'))
        }
    }

    return recur(nums, operators, 0, len(nums)-1)
}

func recur(nums []int, operators []rune, left, right int) (res []int) {
    if left == right {
        res = append(res, nums[left])
        return
    }

    for i := left; i < right; i++ {
        temp1 := recur(nums, operators, left, i)
        temp2 := recur(nums, operators, i + 1, right)

        for _, v1 := range temp1 {
            for _, v2 := range temp2 {
                res = append(res, calc(operators[i], v1, v2))
            }
        }
    }
    return
}

func calc(operator rune, lNum, rNum int) (res int) {
    switch operator {
    case '+' :
        res = lNum + rNum
    case '-' :
        res = lNum - rNum
    case '*' :
        res = lNum * rNum
    }
    return
}
```
