---
title: "[bzoj 2216] [Poi2011]Light Conductor"
date: 2017-07-12 22:36:44
categories:
- bzoj
tags:
- 决策单调性
- 分治
---
一个长度为 n 的序列 a, 对 1 &le; i &le; n, 求最小的自然数 p, 满足对任意 j: a[j] &le; a[i] + p - sqrt(|i-j|). (1 &le; n &le; 5\*10^5, 0 &le; a[i] &le; 10^9)
<!--more-->
类似于 [[bzoj 2687] 交与并](/2017/07/02/yl1706-4-qyh/), 但是简单不少.
$$
ans(i) = \max\left\\{0, \lceil a_j-a_i+\sqrt{|i-j|}\rceil \right\\}
$$
记 $F(i,j) = a_j-a_i+\sqrt{|i-j|}$. 设 $j < k \le i$, 那么
$$
F(j,i) < F(k,i) \Leftrightarrow (a_k-a_j)(\sqrt{i-j} + \sqrt{i-k}) > k-j
$$
所以
$$
F(j,i) < F(k,i) \Rightarrow F(j,i+1) < F(j,i+1)
$$
$F$ 具有凹完全单调性! 又因为我们要最大化答案, 所以决策点非单调右移.

上面为了去绝对值, 假设 $j,k \le i$, 故正反各做一次.

```cpp
const int N = 5e5;
const double inf = 2e9;
int a[N], f[N], g[N];

inline double F(int i, int j)
{
	return a[i] - a[j] + sqrt(j-i);
}

void solve(int* f, int l, int r, int x, int y)
{
	if (l > r) return;
	int m = (l+r)/2, k = -1;
	double b = -inf;
	rep (i, x, min(y, m)+1)
	{
		double t = F(i, m);
		if (t > b)
		{
			b = t;
			k = i;
		}
	}
	f[m] = ceil(b);
	solve(f, l, m-1, x, k);
	solve(f, m+1, r, k, y);
}

int main()
{
	int n;
	scanf("%d", &n);
	rep (i, 0, n) scanf("%d", a+i);
	solve(f, 0, n-1, 0, n-1);
	reverse(a, a+n);
	solve(g, 0, n-1, 0, n-1);
	rep (i, 0, n)
		printf("%d\n", max(0, max(f[i], g[n-i-1])));
	return 0;
}
```