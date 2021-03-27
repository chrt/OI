---
title: "[bzoj 1901] Zju2112 Dynamic Rankings"
date: 2017-02-25 09:20:38
categories:
- bzoj
tags:
- 树套树
- 树状数组
- 线段树
---
题意: 带修改的区间第k小. 询问m, 序列长度n &le; 10^4, 0&le;序列中的数&le;10^9.
<!--more-->
这个数据范围大概分块可以过. 一次询问$O(\lg^3 n)$的算法也可以过, 先前把树状数组套权值线段树的含义理解错了, 外面还套了个二分......

静态区间第k小, 可以把权值线段树持久化, 用对应结点的减法得到某一位置区间的权值线段树. 前缀和是不支持高效修改的, 但是另一个求前缀的数据结构 - 树状数组支持. 所以, 我们维护n棵权值线段树, 第i棵表示位置(i-lowbit(i), i]. 查询的时候仍然在线段树上分治, 维护$O(\lg n)$棵权值线段树中对应的结点, 这样, 就知道了[L, R]中有多少个数属于[l, m). 将其与k比较大小, 使$O(\lg n)$棵线段树同时向左或右儿子走一步. 修改的时候在一系列线段树中原数-1, 新数+1即可. 时间复杂度$O((m+n)\lg^2 n)$. 只开用到的点, 空间复杂度$O((m+n)\lg n\lg (m+n))$, 因为总共$O(m+n)$种数, 每个位置被$O(\lg n)$棵线段树覆盖, 每棵线段树的高是$O(\lg (m+n))$.

```cpp
#include <bits/stdc++.h>
#define Rep(i, a, b) for (int i = (a), i##_ = (b); i < i##_; ++i)
#define For(i, a, b) for (int i = (a), i##_ = (b); i <= i##_; ++i)
#define Down(i, a, b) for (int i = (a), i##_ = (b); i >= i##_; --i)

using namespace std;
const int MAX_N = 1e4, LG_2N = 15, LG_N = 14;

struct Node {
	Node* lc, * rc;
	int s;
} nodes[2*MAX_N*LG_N*(1+LG_2N) + 1], * nil = nodes, * root[MAX_N+1];

typedef vector<Node*> vec;

struct Cmd {
	char op;
	int i, j, k;
} Q[MAX_N];

int top, n, a[MAX_N+1], h[MAX_N*2];

inline int id(int x)
{
	return lower_bound(h, h+top, x) - h;
}

inline int lowbit(int x)
{
	return x & -x;
}

inline void init()
{
	*nil = (Node){nil, nil, 0};
	fill_n(root, n+1, nil);
}

inline Node* new_node()
{
	static Node* cur = nodes + 1;
	*cur = (Node){nil, nil, 0};
	return cur++;
}

void add(Node* &o, int x, int v, int l, int r) // [l, r)
{
	if (o == nil)
		o = new_node();
	if (r-l > 1) {
		int m = (l+r)/2;
		if (x < m)
			add(o->lc, x, v, l, m);
		else
			add(o->rc, x, v, m, r);
	}
	o->s += v;
}

int query(vec& o, vec& p, int k, int l, int r)
{
	while (r-l > 1) {
		int m = (l+r)/2, sz = 0; // [l, m)
		Rep (i, 0, o.size())
			sz += o[i]->lc->s;
		Rep (i, 0, p.size())
			sz -= p[i]->lc->s;
		if (sz >= k) {
			Rep (i, 0, o.size()) o[i] = o[i]->lc;
			Rep (i, 0, p.size()) p[i] = p[i]->lc;
			r = m;
		} else {
			Rep (i, 0, o.size()) o[i] = o[i]->rc;
			Rep (i, 0, p.size()) p[i] = p[i]->rc;
			l = m;
			k -= sz;
		}
	}
	return l;
}

int main()
{
	int m;
	scanf("%d%d", &n, &m);

	For (i, 1, n) {
		scanf("%d", &a[i]);
		h[top++] = a[i];
	}
	Rep (i, 0, m) {
		scanf(" %c", &Q[i].op);
		if (Q[i].op == 'Q')
			scanf("%d%d%d", &Q[i].i, &Q[i].j, &Q[i].k);
		else {
			scanf("%d%d", &Q[i].i, &Q[i].j);
			h[top++] = Q[i].j;
		}
	}
	
	sort(h, h+top);
	top = unique(h, h+top) - h;

	init();
	For (i, 1, n) {
		a[i] = id(a[i]);
		for (int j = i; j <= n; j += lowbit(j))
			add(root[j], a[i], 1, 0, top);
	}

	Rep (i, 0, m) {
		const Cmd& c = Q[i];
		if (c.op == 'Q') {
			vec o, p;
			
			for (int j = c.j; j; j -= lowbit(j))
				o.push_back(root[j]);
			for (int j = c.i-1; j; j -= lowbit(j))
				p.push_back(root[j]);
			
			printf("%d\n", h[query(o, p, c.k, 0, top)]);
		} else {
			int b = id(c.j);
			for (int j = c.i; j <= n; j += lowbit(j)) {
				add(root[j], a[c.i], -1, 0, top);
				add(root[j], b, 1, 0, top);
			}
			a[c.i] = b;
		}
	}

	return 0;
}
```