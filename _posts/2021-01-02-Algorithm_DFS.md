---
layout: post
title: "Algorithm DFS"
date: 2021-01-02 09:00:00 +0800
tags: Algorithm Leetcode
---

记录 SFS 的算法实现

### "39. Combination Sum"

```Go
func combinationSum(candidates []int, target int) (ret [][]int) {
    if len(candidates) == 0 || target == 0 {
        return nil
    }
    for i, v := range candidates {
        if v == target {
            ret = append(ret, []int{target})
        } else if v < target {
            tmpRet := combinationSum(candidates[i:], target-v)
            for _, sli := range tmpRet {
                ret = append(ret, append(sli, v))
            }
        }
    }
    return ret
}
```

### "91. Decode Ways"

```Go
func numDecodings(s string) (ret int) {
    m := map[string]bool{}
    for i := 1; i <= 26; i++ {
        m[strconv.Itoa(i)] = true
    }
    mem := map[int]int{}
    var recur func(string, int) int
    recur = func(s string, idx int) (ret int) {
        if idx == len(s) {
            return 1
        }
        if val, ok := mem[idx]; ok{
            return val
        }
        if m[string(s[idx])] {
            ret += recur(s, idx+1)
        }
        if idx+1 < len(s) && m[string(s[idx:idx+2])] {
            ret += recur(s, idx+2)
        }
        mem[idx] = ret
        return ret
    }
    return recur(s, 0)
}
```

### "297. Serialize and Deserialize Binary Tree"

```Go
// Serializes a tree to a single string.
func (this *Codec) serialize(root *TreeNode) string {
    var builder strings.Builder
    var dfs func(*TreeNode)
    dfs = func(cur *TreeNode) {
        if cur == nil {
            builder.WriteString("")
            return
        }
        builder.WriteByte('(')
        dfs(cur.Left)
        builder.WriteString(fmt.Sprintf(")(%v)(", cur.Val))
        dfs(cur.Right)
        builder.WriteByte(')')
    }
    builder.WriteByte('(')
    dfs(root)
    builder.WriteByte(')')
    ret := builder.String()
    return ret
}

// Deserializes your encoded data to tree.
func (this *Codec) deserialize(data string) *TreeNode {
    if data == "" || data == "()" {
        return nil
    }

    index := 0
    var dfs func() *TreeNode
    dfs = func() *TreeNode {
        if index >= len(data) {
            return nil
        }

        index++
        if data[index] == ')' {
            index++
            return nil
        }
        cur := &TreeNode{}
        cur.Left = dfs()
        index++
        negative := false
        for ; index < len(data) && data[index] != ')'; index++ {
            if data[index] == '-' {
                negative = true
                continue
            }
            cur.Val = cur.Val*10 + int(data[index]-'0')
        }
        if negative {
            cur.Val *= -1
        }
        index++
        cur.Right = dfs()
        index++
        return cur
    }

    return dfs()
}
```

### "377. Combination Sum IV" C++ DFS

```C++
int dfs(vector<int>& dp, int target, vector<int>& nums) {
    if (target == 0)
        return 1;
    if (dp[target] != -1)
        return dp[target];
    int sum = 0;
    for (auto num : nums) {
        if (target >= num) {
            sum += dfs(dp, target - num, nums);
        }
    }
    return dp[target] = sum;
}
int combinationSum4(vector<int>& nums, int target) {
    vector<int> dp(target + 1, -1);
    return dfs(dp, target, nums);
}
```
