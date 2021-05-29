---
layout: post
title: "Algorithm Segment Tree"
date: 2021-05-02 09:00:00 +0800
tags: Algorithm Leetcode
---

记录 Segment Tree 的算法实现

# 数据结构

- ST 的关键优势是和 BIT 一样提供`O(logn)`的`Update`和`RangeSum`功能，并且可以提供`RangeMin/RangeMax`的高级功能。

- ST(Segment Tree 线段树)的基本逻辑是：

  1. 所有**叶子结点存储实际数字**，存储的实际元素范围和 BIT 一样，是一个数组，元素数量是**固定的**
  2. 中间结点存储其孩子结点的`sum/min/max`，同时记录自己所负责的 start、end 范围

- ST 每次更新时和 BIT 逻辑不同，BIT 只能更新 **delta** 值，而 ST 可以更新 **value**，所以 ST **功能更强**
- ST 可以提供`Range 最小公倍数`、`Range 最大公因数`等各种功能
- ST 可以提供`RangeUpdate`功能，经过`Lazy Propogation`优化，时间复杂度可达到`O(logn)`

## 线段树，以结点指针实现(推荐，常用)

![ST tree node](/assets/images/2021-05-02-Algorithm_SegmentTree_1.jpeg)

1. `Build`时，递归完成所有结点的创建，先创建孩子再创建父节点，父节点的`sum`要由左右孩子统计得出，时间复杂度`O(logn)`
2. `Update`时，从上到下递归更新，先更新孩子再更新父节点，向下更新时只更新所属范围内的结点，时间复杂度`O(logn)`
3. `RangeXXX`时，从上到下递归查询，时间复杂度`O(logn)`

## 线段树，以数组实现

用到的比较少，基本思路就是利用二叉树的数组存储法找到孩子和父节点。

## ZKW(张昆伟)线段树，以数组实现(应该理解并记住模板)

zkwST 利用堆式存储，类似 BIT 的方式利用二进制特征快速找到孩子和父节点，实现了**最精简的 SegmentTree**。
适合作为竞赛时的模板，但是真实面试使用可能会因为过于复杂而难以解释。

![zkwST](/assets/images/2021-05-02-Algorithm_SegmentTree_2.jpg)

- 如图，是一个线段树的堆式存储，可以看出**下标**规律：

  1. 父结点下标是当前下标`右移一位`
  2. 当前结点的左子结点是`左移一位`，右子结点是`左移一位+1`
  3. 第`n`层有`2^(n-1)`个结点
  4. 最底层叶子结点数决定了值域

- 用数组(命名为 st)作为**堆式存储**，叶子结点存放实际值，根据完全二叉树的特征，设原值数组长度为`n`，先寻找`2^k >= n`的最小`k`值
  ，找到后有`N = 2^k`，则 st 存储数组长度为`2*N`。
  这里注意到一个细节：完全二叉树叶子结点数`n`和完全二叉树分支结点数`m`的关系应该是`n = m+1`，这里`N`有可能浪费了一些元素，
  目的是在求 RangeSum 时可以利用二进制左右子树查询，所以后面所有循环结束条件都是`i > 0`，下标为`0`的元素没有被使用。
- `build`
  1. 先寻找符合`N = 2^k >= (n+2)`的最小`N`值，创建容量为`2N`的数组
  1. 先将原值数组放入 st 数组最后从`N`开始的`n`个位置，之后注意操作原值(叶子)结点时一定要`i+N`
  1. 然后从位置`N-1`开始倒序循环处理每个父结点进行初始化，其子结点下标正好为`2*i`和`2*i+1`，倒序初始化的过程类似 DP
- `update`
  1. 先要类似 BIT，将 value 转化为 delta
  2. 然后从下到上循环更结点，父结点下标正好为`i/2`或`i>>1`
- `rangeSum`
  1. 左右双指针从下向上遍历，如果一直符合边界，则持续向上；如果有边界外的部分，则先累加"枝叉"；最后累加符合边界的"主干"
  2. 在循环处理指针时有个非常精巧的二进制处理，如下图，如果判断是右子结点命中左边界 l，则`l++`后再循环，这样下一次 l 将指向父节点的右兄弟，
     这样就可以自动排除无效的范围。左子结点命中右边界时同理。

![zkwST](/assets/images/2021-05-02-Algorithm_SegmentTree_3.jpg)

-

```C++
// 参考代码：区间最小值
void Sum(int s,int t,int L=0,int R=0){
    for(s=s+M-1,t=t+M+1;s^t^1;s>>=1,t>>=1){
        L+=d[s],R+=d[t];
        if(~s&1) L=min(L,d[s^1]);
        if(t&1) R=min(R,d[t^1]);
    }
    int res=min(L,R);while(s) res+=d[s>>=1];
}
```

# 题目

### "307. Range Sum Query - Mutable"

```C++
// normal ST
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

```Go
// tree node BST
type NumArray struct {
    stRoot *STNode
}

type STNode struct {
    start, end, sum int
    left, right *STNode
}

func buildTree(start, end int, arr []int) *STNode {
    if start == end {
        return &STNode{
            start: start,
            end: end,
            sum: arr[start],
        }
    }
    mid := (start+end) >> 1
    left := buildTree(start, mid, arr)
    right := buildTree(mid+1, end, arr)
    return &STNode{
        start: start,
        end: end,
        sum: left.sum + right.sum,
        left: left,
        right: right,
    }
}

func (p *STNode) Update(index, value int) {
    if p.start == index && p.end == index {
        p.sum = value
        return
    }
    mid := (p.start+p.end) >> 1
    if index <= mid {
        p.left.Update(index, value)
    } else {
        p.right.Update(index, value)
    }
    p.sum = p.left.sum + p.right.sum
}

func (p *STNode) RangeSum(start, end int) int {
    if p.start == start && p.end == end {
        return p.sum
    }
    mid := (p.start+p.end) >> 1
    if end <= mid {
        return p.left.RangeSum(start, end)
    } else if mid < start {
        return p.right.RangeSum(start, end)
    }
    return p.left.RangeSum(start, mid) + p.right.RangeSum(mid+1, end)
}

func Constructor(nums []int) NumArray {
    return NumArray{stRoot: buildTree(0, len(nums)-1, nums)}
}
func (this *NumArray) Update(index int, val int)  {
    this.stRoot.Update(index, val)
}
func (this *NumArray) SumRange(left int, right int) int {
    return this.stRoot.RangeSum(left, right)
}
```

```C++
// zkwST
class SegmentTree {
    int n;
    int[] st;

    public SegmentTree(int[] nums) {
        n = nums.length; st = new int[2 * n];
        for (int i = n; i < n * 2; i++) st[i] = nums[i-n];
        for (int i = n - 1; i > 0; i--) st[i] = st[2 * i] + st[2 * i + 1];
    }

    public void update(int i, int val) {
        int diff = val - st[i + n];
        for (i += n; i > 0; i /= 2) st[i] += diff;
    }

    public int sumRange(int i, int j) {
        int res = 0;
        for (i += n, j += n; i <= j; i /= 2, j /= 2) {
            if (i % 2 == 1) res += st[i++]; // st[i]是右子结点
            if (j % 2 == 0) res += st[j--]; // st[j]是左子结点
        }
        return res
    }
}
```

```Go
// zkwST
type NumArray struct {
    st []int
    n int
}

func Constructor(nums []int) NumArray {
    m, n := len(nums), 1
    for ; n < m; n <<= 1 {}
    st := make([]int, 2*n)
    copy(st[n:], nums)
    for i := n-1; i > 0; i-- {st[i] = st[i<<1] + st[i<<1 + 1]}
    return NumArray{n: n, st: st}
}

func (this *NumArray) Update(index int, val int)  {
    diff := val - this.st[index+this.n]
    for i := index+this.n; i > 0; i >>= 1 {this.st[i] += diff}
}

func (this *NumArray) SumRange(l int, r int) (ret int) {
    for l,r = l+this.n, r+this.n; l <= r; l,r = l>>1,r>>1 {
        if l & 1 == 1 {
            ret += this.st[l]
            l++
        }
        if r & 1 == 0 {
            ret += this.st[r]
            r--
        }
    }
    return ret
}
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
