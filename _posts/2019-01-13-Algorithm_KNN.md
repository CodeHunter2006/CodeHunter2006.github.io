---
layout: post
title:  "算法学习，机器学习之——K邻近(KNN)"
date:   2019-01-13 9:00:00 +0800
tags: Algorithm
---
### 算法特性
K邻近，KNN(Kth Nearest Neighbor)<br/>
是机器学习常用的分类算法，类似的分类算法还有SVM(支持向量机 Support Vector Machine)。<br/>
基本原理是在n维空间中，距离较近的元素分为一类。

<br/>
### 算法流程
* 数据标准化处理，各个维度的数据转为同一数量级（两种常用方法：线性归一化Min-Max scaling、0均值标准化Z-score standardization）
* 通过新值与样本的距离（欧氏距离/曼哈顿距离/余弦定理）取得最近的K个邻居
* 由K个邻居的类型投票决定，K的取值最好为3~10之间的奇数
* 通过实验观察，调整K的值到最优(训练)

<br/>
### 基本训练过程
* 选定训练集和测试集，一般训练集占70%、测试集占30%
* 对算法进行训练，寻找最合适的K取值
* 通过调整训练集和测试集，优化参数取值(由于KNN是最简单的，一般优化范围有限)

<br/>
### 算法的应用示例
```
/*
Classify the Trade

challenge:
give you old trades and labels, return the new labels of the new trades. 
Trades will be 3 dimension.

solve:
KNN (Kth Nearest Neighbors): 1 get proper k(make sure to be an odd number).
2 calculate every trade euclidean distance with old trades and
keep k-th nearest element in priority queue.
3 count the element in priority queue to get the result.

notice:
code line should not exceed 30.
*/

vector<string> ClassifyTheTrade(vector<vector<float>> &trades,
vector<string> &labels, vector<vector<float>> &newTrades) {
	const int N = trades.size(), M = newTrades.size(), MIN_K = 1, MAX_K = 9;
	int k = 1, greenNum = 0, redNum = 0;
	for (auto l : labels)
		l[0] == 'g' ? greenNum++ : redNum++;
	k = sqrt(min(greenNum, redNum));	// try to get proper k
	k = k&1 ? k : k - 1;	// make sure k to be odd
	k = max(min(k, MAX_K), MIN_K);	// make sure k to be valid
	vector<string> res;
	for (int i = 0; i < M; ++i) {
		priority_queue<pair<float,int>> pq;
		for (int j = 0; j < N; ++j) {
			float distance = sqrt(pow(newTrades[i][0] - trades[j][0], 2) +
					pow(newTrades[i][1] - trades[j][1], 2) +
					pow(newTrades[i][1] - trades[j][1], 2));
			pq.push(make_pair(distance, labels[j][0] == 'g' ? 1 : -1));
			while (pq.size() > k)
				pq.pop();
		}
		int resType = 0;
		while (pq.size()) {
			resType += pq.top().second;
			pq.pop();
		}
		if (resType > 0)
			res.push_back("green");
		else
			res.push_back("red");
	}
	return res;
}
```
