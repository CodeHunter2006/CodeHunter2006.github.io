---
layout: post
title:  "算法学习，极限数据结构之——索引树(BIT)"
date:   2018-12-09 10:00:00 +0800
tags: Algorithm
---
### 算法特性
树状数组，索引树，BIT(Binary Indexed Tree)<br/>
提供getSum(i)和update(i,val)函数。<br/>
可以获取目标数组第i位置(包括)往前所有元素的和(可以求和也可以求范围内最大或对小值)，<br/>
可以动态更新某位置的值。getSum和update的时间复杂度都是O(logn)。<br/>
如果用一般的遍历方法求sum，时间复杂度是O(n)，更新一个值的时间复杂度是O(1)；<br/>
如果我们用dp sum的方式来实现sum的快速查询，那么查询的时间复杂度是O(1)，但更新一个值的时间复杂度是O(n)相比较O(logn)较差。<br/>
使用BIT适用于查询和更新频繁度相当的情况，可以降低总的时间复杂度。

<br/>
### 数据结构的实现
```
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
```

<br/>
### 应用示例
```
// 暂时缺少
```

