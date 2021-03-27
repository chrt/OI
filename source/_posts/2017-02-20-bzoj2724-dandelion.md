---
title: "[bzoj 2724] [Violet 6] 蒲公英"
date: 2017-02-20 15:23:55
categories:
- bzoj
tags:
- 分块
- 众数
---
题意: 给定长度为n的序列和q个询问, 每次询问某区间的众数, 如有多个, 取最小者, 强制在线. (n &le; 4\*10^4, q &le; 5\*10^4)

<!--more-->
[区间众数](/2017/02/20/segment-mode/)

写了一发$O((n+q)\sqrt n)$, 然而常数巨大......

```cpp
#include <cstdio>
#include <cmath>
#include <algorithm>
#define fst first
#define sec second
using namespace std;
typedef pair<int, int> ii;
const int MAX_N = 4e4, SIZE = 200, BLOCK = MAX_N/SIZE;
int n, N, sz, blk, x[MAX_N], h[MAX_N], C[BLOCK][MAX_N], A[MAX_N][SIZE], id[BLOCK][MAX_N], t[MAX_N];
ii f[BLOCK][BLOCK];

inline void update(ii& x, const ii& y)
{
	x = y.fst > x.fst || y.fst == x.fst && y.sec < x.sec ? y : x;
}

void init()
{
	sz = sqrt(n);
	blk = (n+sz-1)/sz;

	// f[i][j]: 块i到块j的众数
	for (int i = 0; i < blk; ++i) {
		ii ans(0, 0);
		for (int j = i; j < blk; ++j) {
			int k0 = j*sz, l = min(n, k0+sz);
			for (int k = k0; k < l; ++k)
				++t[x[k]];
			for (int k = k0; k < l; ++k)
				update(ans, ii(t[x[k]], x[k]));
			f[i][j] = ans;
		}
		fill_n(t, N, 0);
	}
	
	// C[i][x]: 块0~i中x的出现次数
	for (int i = 0, j = 0; i < blk; ++i)
		for (int k = min(j+sz, n); j < k; ++j)
			++C[i][x[j]];
	for (int i = 1; i < blk; ++i)
		for (int j = 0; j < N; ++j)
			C[i][j] += C[i-1][j];

	// A[i][x]: i所在块开始位置~i位置 x的出现次数
	// id[i][x]: 块i中x的索引
	for (int i = 0, j = 0; i < blk; ++i) {
		fill_n(id[i], N, -1);
		for (int k = min(j+sz, n), c = 0; j < k; ++j) {
			int& l = id[i][x[j]];
			l = l == -1 ? c++ : l;
			++A[j][l];
		}
	}
	for (int i = 0, j = 1; i < blk; ++i, ++j)
		for (int k = min(j-1+sz, n); j < k; ++j)
			for (int l = 0; l < sz; ++l)
				A[j][l] += A[j-1][l];
}

inline int cnt(int a, int p)
{
	int b = p/sz, i = id[b][a];
	return (b ? C[b-1][a] : 0) + (i >= 0 ? A[p][i] : 0);
}

inline int cnt(int a, int l, int r)
{
	return cnt(a, r) - (l ? cnt(a, l-1) : 0);
}

int query(int l, int r)
{
	int lb = l/sz, rb = r/sz;
	ii ans(0, 0);
	if (lb == rb) {
		for (int i = l; i <= r; ++i)
			++t[x[i]];
		for (int i = l; i <= r; ++i)
			update(ans, ii(t[x[i]], x[i]));
		for (int i = l; i <= r; ++i)
			t[x[i]] = 0;
	} else {
		ans = rb-lb > 1 ? f[lb+1][rb-1] : ans;
		for (int i = l, st = (lb+1)*sz; i < st; ++i)
			update(ans, ii(cnt(x[i], l, r), x[i]));
		for (int i = rb*sz; i <= r; ++i)
			update(ans, ii(cnt(x[i], l, r), x[i]));
	}
	return ans.sec;
}

int main()
{
	int m;
	scanf("%d %d", &n, &m);
	for (int i = 0; i < n; ++i) {
		scanf("%d", &x[i]);
		h[i] = x[i];
	}
	sort(h, h+n);
	N = unique(h, h+n) - h;
	for (int i = 0; i < n; ++i)
		x[i] = lower_bound(h, h+N, x[i]) - h;
	init();
	int last = 0;
	while (m--) {
		int l, r;
		scanf("%d %d", &l, &r);
		l = (l + last - 1) % n;
		r = (r + last - 1) % n;
		if (l > r)
			swap(l, r);
		printf("%d\n", last = h[query(l, r)]);
	}
	return 0;
}
```