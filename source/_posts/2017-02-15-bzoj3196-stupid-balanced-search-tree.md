---
title: "[bzoj 3196] Tyvj 1730 二逼平衡树：线段树套平衡树"
date: 2017-02-15 15:59:30
categories:
- 模板
tags:
- 树套树
- 线段树
- Treap
---

一种支持查询区间排名、区间k小、区间前驱、区间后继和单点修改的数据结构。
<!--more-->
如果知道这是线段树套平衡树，写起来还是挺容易的，就是有点无聊......

为了确保二分的时间复杂度，离散化了一下。

```cpp
{% raw %}
#include <cstdio>
#include <algorithm>
#include <cstdlib>
#include <cmath>
using namespace std;

const int MAX_N = 1e5, MAX_D = 16, MAX_M = 1e5;

int n, N;

namespace Treap {
	struct Node {
		Node* ch[2];
		int v, s, num, r;
		void up()
		{
			s = ch[0]->s + ch[1]->s + num;
		}
	} nodes[MAX_N*(MAX_D+1) + 1] = {(Node){{0, 0}, 0, 0, 0, -1}}, * null = nodes;

	inline Node* new_node(int v)
	{
		static Node* cur = nodes + 1;
		*cur = (Node){{null, null}, v, 1, 1, rand()};
		return cur++;
	}

	inline void rotate(Node* &x, int d)
	{
		Node* y = x->ch[d];
		x->ch[d] = y->ch[d^1];
		y->ch[d^1] = x;
		x->up();
		y->up();
		x = y;
	}

	void insert(Node* &x, int v)
	{
		if (x == null)
			x = new_node(v);
		else if (x->v == v)
			++x->num, ++x->s;
		else {
			int d = v > x->v;
			insert(x->ch[d], v);
			if (x->ch[d]->r > x->r)
				rotate(x, d);
			else
				x->up();
		}
	}

	void remove(Node* &x, int v)
	{
		if (x->v == v) {
			if (x->num > 1)
				--x->num, --x->s;
			else if (x->ch[0] == null)
				x = x->ch[1];
			else if (x->ch[1] == null)
				x = x->ch[0];
			else {
				int d = x->ch[1]->r > x->ch[0]->r;
				rotate(x, d);
				remove(x->ch[d^1], v);
				x->up();
			}
		} else {
			remove(x->ch[v > x->v], v);
			x->up();
		}
	}
	
	inline void modify(Node* &x, int u, int v)
	{
		remove(x, u);
		insert(x, v);
	}

	int find(Node* x, int v, int d)
	{
		Node* y = null;
		while (x != null) {
			if (x->v != v && x->v > v == d) {
				y = x;
				x = x->ch[d^1];
			} else
				x = x->ch[d];
		}
		return y == null ? -1 : y->v;
	}
	
	// 小于 v 的数的个数
	int count(Node* x, int v)
	{
		int c = 0;
		while (x != null) {
			if (v == x->v)
				return c + x->ch[0]->s;
			if (v > x->v) {
				c += x->num + x->ch[0]->s;
				x = x->ch[1];
			} else
				x = x->ch[0];
		}
		return c;
	}
}

namespace Segment_Tree {
	Treap::Node* root[MAX_N*4];

	int count(int v, int L, int R, int o, int l, int r)
	{
		if (L <= l && r <= R)
			return Treap::count(root[o], v);
		int m = (l+r)/2, ans = 0;
		if (L <= m)
			ans = count(v, L, R, o*2, l, m);
		if (R > m)
			ans += count(v, L, R, o*2+1, m+1, r);
		return ans;
	}

	inline int rank(int v, int L, int R)
	{
		return count(v, L, R, 1, 1, n) + 1;
	}

	// rank 小于等于 k 的最大数
	int kth(int k, int L, int R)
	{
		int l = 0, r = N; // [l, r)
		while (r-l > 1) {
			int m = (l+r)/2, rk = rank(m, L, R);
			if (rk <= k)
				l = m;
			else
				r = m;
		}
		return l;
	}
	
	void modify(int pos, int u, int v, int o, int l, int r)
	{
		Treap::modify(root[o], u, v);
		if (l != r) {
			int m = (l+r)/2;
			if (pos <= m)
				modify(pos, u, v, o*2, l, m);
			else
				modify(pos, u, v, o*2+1, m+1, r);
		}
	}

	int query(int v, int d, int L, int R, int o, int l, int r)
	{
		if (L <= l && r <= R) {
			int t = Treap::find(root[o], v, d);
			return t == -1 ? (d ? N : 0) : t;
		}
		int m = (l+r)/2, ans = d ? N : 0;
		if (L <= m)
			ans = query(v, d, L, R, o*2, l, m);
		if (R > m) {
			int t = query(v, d, L, R, o*2+1, m+1, r);
			ans = t < ans == d ? t : ans;
		}
		return ans;
	}

	void build(int A[], int o, int l, int r)
	{
		root[o] = Treap::null;
		for (int i = l; i <= r; ++i)
			Treap::insert(root[o], A[i]);
		if (l != r) {
			int m = (l+r)/2;
			build(A, o*2, l, m);
			build(A, o*2+1, m+1, r);
		}
	}
}

struct Cmd {
	int opt, fst, sec, thd;
} C[MAX_M];

int A[MAX_N + 1], H[MAX_N + MAX_M];

inline int idx(int x)
{
	return lower_bound(H, H+N, x) - H;
}

int main()
{
	srand(65535);
	int m;
	scanf("%d %d", &n, &m);
	for (int i = 1; i <= n; ++i) {
		scanf("%d", &A[i]);
		H[N++] = A[i];
	}
	for (int i = 0; i < m; ++i) {
		scanf("%d", &C[i].opt);
		if (C[i].opt == 3) {
			scanf("%d %d", &C[i].fst, &C[i].sec);
			H[N++] = C[i].sec;
		} else {
			scanf("%d %d %d", &C[i].fst, &C[i].sec, &C[i].thd);
			if (C[i].opt != 2)
				H[N++] = C[i].thd;
		}
	}
	sort(H, H+N);
	N = unique(H, H+N) - H;
	for (int i = 1; i <= n; ++i)
		A[i] = idx(A[i]);
	using namespace Segment_Tree;
	build(A, 1, 1, n);
	for (int i = 0; i < m; ++i) {
		int o = C[i].opt, a = C[i].fst, b = C[i].sec, c = C[i].thd, v;
		switch (o) {
			case 1:
				printf("%d\n", rank(idx(c), a, b));
				break;
			case 2:
				printf("%d\n", H[kth(c, a, b)]);
				break;
			case 3:
				modify(a, A[a], v = idx(b), 1, 1, n);
				A[a] = v;
				break;
			default:
				printf("%d\n", H[query(idx(c), o == 5, a, b, 1, 1, n)]);
		}
	}
	return 0;
}
{% endraw %}
```