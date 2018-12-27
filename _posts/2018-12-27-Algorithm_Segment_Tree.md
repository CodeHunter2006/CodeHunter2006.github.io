---
layout: post
title:  "算法学习，极限数据结构之——线段树(Segment Tree)"
date:   2018-12-27 10:00:00 +0800
tags: Algorithm
---
### 算法特性
线段树，Segment Tree<br/>
与之前介绍的索引树BIT功能和性能基本相同，适用于对所有元素求和并且查询和更新频度相当的情况。<br/>
更新和求和的时间复杂度都为O(n)。

<br/>
### 数据结构的实现
```
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
```

<br/>
### 应用示例
```
// 暂时缺少
```

