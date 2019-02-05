---
layout: post
title:  "算法学习，机器学习之——退火算法(Annealing)"
date:   2019-02-05 10:00:00 +0800
tags: Algorithm
---
### 算法特性
退火，Annealing<br/>
利用微积分的思路，通过检测、迭代、缩减Delta，最终取得接近最优值的结果。<br/>

<br/>
### 算法流程
* 设定目标值与运算结果的可接受的期望偏差EPS(Expects)、每次缩减变动量的缩减系数DELTA、设定出发点坐标
* 对每个维度围绕当前点前后做偏差检测，如果距离目标值更近则保留，否则抛弃
* 如果在当前偏差量下无法继续靠近，则缩小偏差量，逐步靠近
* 最终得到符合EPS的结果

<br/>
### 算法的应用示例
```
/*
  下面的算法可以实现对三角形求费马点(Fermat Point)，即在三角形内距离三个顶点的距离和最小的点。
*/

// get the sum distance between the given point to three triangle points.
double getDistance(const vector<pair<double,double>> &vertexes,
 pair<double,double> point) {
	double distSum = 0;
	for (auto v : vertexes) {
		distSum += sqrt(pow(v.first - point.first, 2.0) 
		+ pow(v.second - point.second, 2.0));
	}
	return distSum;
}

// get Fermat Point
pair<float,float> IntegralPointInsideATriangle(pair<double,double> p1,
 pair<double,double> p2, pair<double,double> p3) {
	const double EPS = 1e-6, DELTA = 0.98;

	double t = 100.0, distance = DBL_MAX;
	
	// 下面的代码增加了几个多余的'\'，为了在jekyll中通过编译，实际使用时要去掉
	vector<vector<double>> dir({\{0,1},\{0,-1},\{1,0},\{-1,0}});
	pair<double,double> resPoint = p1;
	vector<pair<double,double>> vertexes({p1,p2,p3});

	while (t > EPS) {
		bool go = true;
		while (go) {
			go = false;
			auto curPoint = resPoint;
			auto curDistance = distance;

			for (auto d : dir) {
				auto newPoint = resPoint;
				newPoint.first += t * d[0];
				newPoint.second += t * d[1];

				float newDist = getDistance(vertexes, newPoint);
				if (newDist < curDistance) {
					curDistance = newDist;
					curPoint = newPoint;
				}
			}
			if (curDistance < distance) {
				go = true;
				distance = curDistance;
				resPoint = curPoint;
			}
		}
		t *= DELTA;
	}

	return resPoint;
}
```

<br/>
### 改进
* 退火算法可以通过梯度下降方法，更快的向目标逼近，这样在大规模计算时候，可以避免很多不必要的运算。
* 退火算法可能陷入局部最优解，可以在计算过程中添加一个随机探索过程，避免陷入局部最优解而全局较差解。
