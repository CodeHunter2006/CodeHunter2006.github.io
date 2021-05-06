---
layout: post
title: "Algorithm Segment Tree"
date: 2021-05-02 09:00:00 +0800
tags: Algorithm Leetcode
---

记录 Segment Tree 的算法实现

### "307. Range Sum Query - Mutable"

```C++
class NumArray {
private:
    class SegmentNode {
        public:
        SegmentNode(int start, int end, int sum, SegmentNode* left = nullptr, SegmentNode* right = nullptr) :
        start(start), end(end), sum(sum), left(left), right(right){}
        SegmentNode(const SegmentNode&) = delete;
        SegmentNode& operator=(const SegmentNode&) = delete;
        ~SegmentNode() {
            delete left;
            delete right;
            left = right = nullptr;
        }
        int start;
        int end;
        int sum;
        SegmentNode* left;
        SegmentNode* right;
    };
    SegmentNode* buildTree(int start, int end, vector<int> &vals) {
        if (start == end)
            return new SegmentNode(start, end, vals[start]);
        int mid = (end + start)>>1;
        auto left = buildTree(start, mid, vals);
        auto right = buildTree(mid + 1, end, vals);
        return new SegmentNode(start, end, left->sum + right->sum, left, right);
    }
    void updateTree(SegmentNode* root, int index, int value) {
        if (root->start == index && root->end == index) {
            root->sum = value;
            return;
        }
        int mid = (root->start + root->end)>>1;
        if (index <= mid) {
            updateTree(root->left, index, value);
        } else {
            updateTree(root->right, index, value);
        }
        root->sum = root->left->sum + root->right->sum;
    }
    int sumRange(SegmentNode* root, int start, int end) {
        if (root->start == start && root->end == end)
            return root->sum;
        int mid = (root->start + root->end)>>1;
        if (end <= mid) {
            return sumRange(root->left, start, end);
        } else if (mid < start) {
            return sumRange(root->right, start, end);
        } else {
            return sumRange(root->left, start, mid) + sumRange(root->right, mid + 1, end);
        }
    }
    std::unique_ptr<SegmentNode> root_;
public:
    NumArray(vector<int>& nums) {
        if (!nums.empty())
        	root_.reset(buildTree(0, nums.size() - 1, nums));
    }

    void update(int i, int val) {
        updateTree(root_.get(), i, val);
    }

    int sumRange(int i, int j) {
        return sumRange(root_.get(), i, j);
    }
};
```

### "850. Rectangle Area II"

```C++
class Solution {
private:
    class SegmentNode {
        public:
        long total;
        SegmentNode(int start, int end, vector<int> &xArray) :
        start_(start), end_(end), xArray_(xArray), left_(nullptr), right_(nullptr),
        count_(0), total(0) {
        }
        SegmentNode(const SegmentNode&) = delete;
        SegmentNode& operator=(const SegmentNode&) = delete;
        ~SegmentNode() {
            delete left_;
            delete right_;
            left_ = right_ = nullptr;
        }

        SegmentNode* getLeft() {
            if (left_ == nullptr)
                left_ = new SegmentNode(start_, getRangeMid(), xArray_);
            return left_;
        }
        SegmentNode* getRight() {
            if (right_ == nullptr)
                right_ = new SegmentNode(getRangeMid(), end_, xArray_);
            return right_;
        }
        long update(int start, int end, int val) {
            if (start >= end) return 0;
            if (start_ == start && end_ == end) {
                count_ += val;
            } else {
                getLeft()->update(start, min(getRangeMid(), end), val);
                getRight()->update(max(getRangeMid(), start), end, val);
            }
            if (count_ > 0)
                total = xArray_[end_] - xArray_[start_];
            else
                total = getLeft()->total + getRight()->total;
            return total;
        }

        private:
        int start_;
        int end_;
        vector<int> &xArray_;
        int count_;
        SegmentNode* left_;
        SegmentNode* right_;

        int getRangeMid() {
            return (start_ + end_)>>1;
        }
    };
public:
    int rectangleArea(vector<vector<int>>& rectangles) {
        if (rectangles.empty() || rectangles[0].empty()) return 0;
        const int N = rectangles.size(), OPEN = 1, CLOSE = -1;
        vector<vector<int>> events(N * 2, vector<int>(4));
        unordered_set<int> xSet;
        for (int i = 0; i < N; ++i) {
            events[2 * i] = {rectangles[i][1], OPEN, rectangles[i][0], rectangles[i][2]};
            events[2 * i + 1] = {rectangles[i][3], CLOSE, rectangles[i][0], rectangles[i][2]};
            xSet.insert(rectangles[i][0]);
            xSet.insert(rectangles[i][2]);
        }
        sort(events.begin(), events.end());
        vector<int> xArray(xSet.begin(), xSet.end());
        sort(xArray.begin(), xArray.end());
        unordered_map<int,int> x2i;
        for (int i = 0; i < xArray.size(); ++i)
            x2i[xArray[i]] = i;
        std::unique_ptr<SegmentNode> activeTree(new SegmentNode(0, xArray.size(), xArray));
        int curY = events[0][0];
        long ans = 0, curXSum = 0;
        for (auto &e : events) {
            int y = e[0], type = e[1], x1 = e[2], x2 = e[3];
            ans += curXSum * (y - curY);
            curXSum = activeTree->update(x2i[x1], x2i[x2], type);
            curY = y;
        }
        ans %= 1000000007;
        return ans;
    }
};
```
