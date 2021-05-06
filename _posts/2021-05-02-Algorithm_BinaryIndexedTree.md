---
layout: post
title: "Algorithm Binary Indexed Tree"
date: 2021-05-02 09:00:00 +0800
tags: Algorithm Leetcode
---

记录 Binary Indexed Tree 的算法实现

### "307. Range Sum Query - Mutable"

```C++
class BIT {
    private:
    vector<int> bitree_;
    public:
    BIT (int maxNum) {
        bitree_.resize(maxNum + 1);
    }
    int getSum(int index) {
        int sum = 0;
        index++;
        while (index > 0) {	// 注意这里是 > 不是 >=，否则会造成死循环
            sum += bitree_[index];
            index -= index & (-index);
        }
        return sum;
    }
    void update(int index, int val) {
        index++;
        while (index < bitree_.size()) {
            bitree_[index] += val;
            index += index & (-index);
        }
    }
};
unique_ptr<BIT> pBIT_; // practice unique_ptr
vector<int> &nums_;
NumArray(vector<int>& nums) : nums_(nums) {
    pBIT_ = unique_ptr<BIT>(new BIT(nums.size()));
    for (int i = 0; i < nums.size(); ++i)
        pBIT_->update(i, nums[i]);
}
void update(int i, int val) {
    int diff = val - nums_[i];
    pBIT_->update(i, diff);
    nums_[i] = val;
}
int sumRange(int i, int j) {
    return pBIT_->getSum(j) - pBIT_->getSum(i - 1);
}
```

### "315. Count of Smaller Numbers After Self"

```C++
class BIT {
    public:
    BIT(int size) {
        nums_.resize(size + 1);
    }
    BIT(const BIT&) = delete;
    BIT& operator=(const BIT&) = delete;
    ~BIT() = default;
    void update(int index, int val = 1) {
        index++;
        while (index < nums_.size()) {
            nums_[index] += val;
            index += index&(-index);
        }
    }
    int sum(int index) {
        index++;
        int sum = 0;
        while (index > 0) {
            sum += nums_[index];
            index -= index&(-index);
        }
        return sum;
    }

    private:
    vector<int> nums_;
};
unordered_map<int,int> convert2i(vector<int> &nums) {
    unordered_set<int> numSet;
    for (auto i : nums)
        numSet.insert(i);
    vector<int> numVector(numSet.begin(), numSet.end());
    sort(numVector.begin(), numVector.end());
    unordered_map<int,int> num2i;
    for (int i = 0; i < numVector.size(); ++i)
        num2i[numVector[i]] = i;
    return num2i;
}
vector<int> countSmaller(vector<int>& nums) {
    if (nums.empty()) return {};
    const int N = nums.size();
    vector<int> res(N);
    unordered_map<int,int> num2i = convert2i(nums);
    BIT bit(num2i.size());
    for(int i = N - 1; i >= 0; --i) {
        res[i] = bit.sum(num2i[nums[i]] - 1);
        bit.update(num2i[nums[i]]);
    }
    return res;
}
```

### "493. Reverse Pairs"

```C++
int res = 0, n = nums.size();
BIT bit(n + 1);
vector<int> v = nums;
sort(v.begin(), v.end());
unordered_map<int, int> m;
for (int i = 0; i < n; ++i)
    m[v[i]] = i + 1;
for (int i = n - 1; i >= 0; --i) {
    res += bit.getSum(lower_bound(v.begin(), v.end(), nums[i] / 2.0) - v.begin());
    bit.update(m[nums[i]]);
}
return res;
```
