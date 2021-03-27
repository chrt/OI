---
title: "[APIO 2014] Split the sequence"
date: 2017-05-01 12:26:56
categories:
- APIO
tags:
- 动态规划
- 斜率优化
---
把一个长度为n的非负整数序列ai分成(k+1)个非空的块, 每次分割的得分等于被分割的两部分的元素和的乘积. 求最大总得分, 并输出任意一种方案. (2≤n≤10^5,1≤k≤min{n−1,200},0≤ai≤10^4)
<!--more-->
首先, 有这样的DP: `dp[l][i][j]`表示[i,j]分割l次的最大得分.

然后, 我在想, 要么可以优化, 要么可以贪心. ai是非负数, 可能有什么用处. 如果要对DP进行优化: 1. 就算转移的复杂度降下来, 状态数也太多了. 2. 斜率优化不适用于这种形式. 难道分割的得分与顺序无关? 我甚至没试一试就否定了这个想法......贪心的话, 联想到均值不等式, 但好像没什么用.

设序列被分割成(k+1)块, 每块的元素之和依次为: b[0], b[1], ..., b[k], 则得分为: b[0](b[1]+b[2]+...+b[n]) + b[1](b[2]+b[3]+...+b[n]) + ... + b[n-1](b[n]). 考虑i, j两块. 将它们分开的那一次切割, 它俩相乘; 此前, 此后, 均不会对得分产生贡献. 这个和式说明, 得分与顺序无关. 所以, 状态简化为`dp[i][j]`, 表示[0,j]分割i次的最大得分.

枚举j=1,2,...,k, 每一层斜率优化. ai是非负数, 使得决策点的横坐标和直线的斜率均单调, 用一个队列维护即可.

从数据范围可以看出, 大概得设计时间复杂度O(nk)的算法. 如果要DP, 分割的次数肯定不能从状态中去掉, 很明显有必要考察一下区间是否可以简化成一维嘛......TAT

```cpp
#define x(j) S[j]
#define y(j) dp[i-1][j]

typedef long long ll;

const int N = 1e5, K = 200;

ll dp[K+1][N], S[N];
int pre[K+1][N], Q[N];

inline ll cross(ll x1, ll y1, ll x2, ll y2)
{
	return x1*y2 - x2*y1;
}

int main()
{
	int n, k;
	scanf("%d%d", &n, &k);
	rep (i, 0, n)
		scanf("%lld", S+i);
	rep (i, 1, n)
		S[i] += S[i-1];
	rep (i, 0, n)
		dp[0][i] = S[i] * (S[n-1]-S[i]);

	rep (i, 1, k+1) {
		int front = 0, rear = 0;
		Q[rear++] = i-1;
		rep (j, i, n) {
			int a = Q[front];
			while (rear - front > 1
				   && cross(x(Q[front+1])-x(a), y(Q[front+1])-y(a), 1, S[n-1]-S[j]) <= 0)
				a = Q[++front];
			pre[i][j] = a;
			dp[i][j] = dp[i-1][a] + (S[j]-S[a]) * (S[n-1]-S[j]);

			a = Q[rear-1];
			while (rear - front > 1
				   && cross(x(a)-x(Q[rear-2]), y(a)-y(Q[rear-2]), x(j)-x(a), y(j)-y(a)) >= 0)
				a = Q[--rear - 1];
			Q[rear++] = j;
		}
	}

	printf("%lld\n", dp[k][n-1]);
	
	int p = n-1;
	per (i, k, 1)
		printf("%d ", (p = pre[i][p]) + 1);
	puts("");
	
	return 0;
}
```