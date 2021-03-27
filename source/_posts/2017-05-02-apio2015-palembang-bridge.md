---
title: "[APIO 2015] Palembang Bridges"
date: 2017-05-02 17:33:46
categories:
- APIO
tags:
- 数学
- STL
---
2\*(10^9+1)的网格上有n对点, 横向连通, 纵向可造不多于k座竖直方向的桥, 仅在有桥的位置可以通行. 求每对点最短距离之和的最小值. (k=1或2, 1≤n≤100000)
<!--more-->
先处理掉两点均在同一行的点对, 然后考虑其他点. 加上过河的纵向距离, 网格简化为1行.

有这样一个结论: 关于x的函数y=|x-a[1]|+|x-a[2]|+...+|x-a[n]|, 当x=a[i]的中位数时取得最小值.

证明: 不妨设a[1]≤a[2]≤...≤a[n], 某点x满足a[i]≤x≤a[i+1]. 令x在[a[i],a[i+1]]中移动, 向右移动距离d, 会使y增加d(i-(n-i))=d(2i-n). 将该点从负无穷移向正无穷. 当i&lt;n/2时, y持续减小; 当i&gt;n/2时, y持续增大. 注意i是正整数. 由此看出x∈[floor(n/2), ceil(n/2)]时, y取得最小值.

k=1时, 取出所有点的横坐标, 其中位数即桥的位置. 

k=2时, 由一次函数的性质可知, 最小值在某个数据点处取得, 暴力枚举, 获得9分. 想到一个O(n^2)的做法, 刚刚发现是错的 TAT

失误在于只是草草画图而没有定量分析两点间最短路的走法.

设两点为s,t, 两座桥为a,b (s&le;t, a&lt;b). 可以发现, 若s在a左侧, 走a桥; 若t在b右侧, 走b桥. 当a&le;s&le;t&le;b, 当(s+t)/2&le;(a+b)/2时走a桥, 否则走b桥, 这个条件对任意s&le;t都是适用的. 如果把所有点对按中点排序, 则桥的选择具有了单调性 - 前面全选a, 后面全选b, 无论桥在哪里 (也许有一些点对既可以选择a桥, 又可以选择b桥, 但我们仍然按照(s+t)/2为它选择).

排序后, 枚举决策的分界点, 则左右变成了两个k=1的问题. 用set动态维护中位数 - 每新加两个点, 中位数要么不变, 要么左移或右移一个位置.

```cpp
#define iter iterator

typedef long long ll;
typedef pair<int, int> ii;
vector<ii> X;
vector<ll> Y[2];

template <typename Iter>
void cal(Iter s, Iter t, vector<ll>& v)
{
	set<ii> S;
	ll ans = s->second - s->first;
	int num = 0;
	S.insert(ii(s->first, num++));
	S.insert(ii(s->second, num++));
	set<ii>::iter m = S.begin();
	v.push_back(ans);

	while (++s != t) {
		int x = s->first, y = s->second;
		ii a(x, num++), b(y, num++);
		S.insert(a);
		S.insert(b);
		if (b < *m) {
			int p = m->first, z = (--m)->first;
			ans += 2*(p - z) + (z - x) + (z - y);
		} else if (a > *m) {
			int z = (++m)->first;
			ans += (x - z) + (y - z);
		} else {
			ans += y - x;
		}
		v.push_back(ans);
	}
}

bool cmp(const ii& x, const ii& y)
{
	return x.first + x.second < y.first + y.second;
}

int main()
{
	int k, n;
	scanf("%d%d", &k, &n);

	ll ans = 0;
	rep (i, 0, n) {
		int s, t;
		char p, q;
		scanf(" %c %d %c %d", &p, &s, &q, &t);
		if (p == q) {
			ans += abs(s-t);
		} else {
			++ans;
			if (s > t) swap(s, t);
			X.push_back(ii(s, t));
		}
	}

	int s = X.size();

	if (s) {
		sort(X.begin(), X.end(), cmp);
		cal(X.begin(), X.end(), Y[0]);

		if (k == 1)
			return printf("%lld\n", ans + Y[0][s-1]), 0;

		cal(X.rbegin(), X.rend(), Y[1]);

		ll m = min(Y[0][s-1], Y[1][s-1]);
		rep (i, 0, s-1)
			m = min(m, Y[0][i] + Y[1][s-2-i]);
		ans += m;
	}

	printf("%lld\n", ans);

	return 0;
}
```