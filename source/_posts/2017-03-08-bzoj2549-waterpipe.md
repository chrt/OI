---
title: "[bzoj 2549] [Wc2006]水管局长数据加强版"
date: 2017-03-08 08:25:14
categories:
- bzoj
tags:
- LCT
- 生成树
---
N个点M条边的简单无向图, Q个操作, 每个操作断掉一条边, 或询问两点间路径最大值的最小值. (N&le;10^5, M&le;10^6, Q&le;10^5, 保证任意时刻图连通)
<!--more-->
离线, 把操作倒过来, 转删边为加边, 则本题转化为动态最小生成树问题 (一个图最小生成树上的路径最小化最大权值). LCT, Splay中每个点额外维护子树中权值最大的点的指针. 由于这里是边权, 所以需要加额外的点转化为点权. 新加入一条边(u,v), 比较原来最小生成树上u, v之间路径的最大权值, 如若替换后更优, 则替换之.

初始化的时候有一堆从未被删除的边. 对它们跑一遍Kruskal (像后面一样动态处理据说会被卡).

去年NOI第一试的前一天晚上还在调这道题......现在还是没看出为什么过不了样例, 索性不看了. 所以说不要把代码写得太混乱......

```cpp
#include <bits/stdc++.h>
#define Rep(i, a, b) for (int i = (a), i##_ = (b); i < i##_; ++i)
#define Down(i, a, b) for (int i = (a), i##_ = (b); i >= i##_; --i)

using namespace std;
const int MAX_N = 1e5, MAX_M = 1e6, MAX_Q = 1e5;

int n, m;

struct Node {
	Node* ch[2], * fa, * mx;
	int t, v;
	bool rev;

	Node();
	
	void reverse()
	{
		rev = !rev;
		swap(ch[0], ch[1]);
		ch[0]->t = 0;
		ch[1]->t = 1;
	}
	
	void up()
	{
		mx = this;
		if (ch[0]->mx->v > v)
			mx = ch[0]->mx;
		if (ch[1]->mx->v > mx->v)
			mx = ch[1]->mx;
	}

	void down()
	{
		if (rev) {
			rev = false;
			ch[0]->reverse();
			ch[1]->reverse();
		}
	}

	void setf(Node* f, int t1)
	{
		fa = f;
		t = t1;
		t >= 0 ? f->ch[t] = this : 0;
	}
} nodes[1 + MAX_N + MAX_M], * nil = nodes;

Node::Node(): fa(nil), mx(this), t(-1), v(-1), rev(false)
{
	ch[0] = ch[1] = nil;
}

inline void rot(Node* y)
{
	Node* x = y->fa;
	int t = y->t;
	y->setf(x->fa, x->t);
	y->ch[t^1]->setf(x, t);
	x->setf(y, t^1);
	x->up();
}

void splay(Node* x)
{
	static Node* S[MAX_N + MAX_M];
	int top = 0;
	Node* y;
	for (y = x; y->t >= 0; y = y->fa) S[top++] = y;
	for (y->down(); top; S[--top]->down()) ;

	while (x->t >= 0) {
		if ((y = x->fa)->t >= 0)
			rot(y->t ^ x->t ? x : y);
		rot(x);
	}
	x->up();
}

inline void access(Node* x)
{
	for (Node* y = nil; x != nil; y = x, x = x->fa) {
		splay(x);
		x->ch[1]->t = -1;
		y->setf(x, 1);
		x->up();
	}
}

inline void make_root(Node* x)
{
	access(x);
	splay(x);
	x->reverse();
}

inline void link(Node* x, Node* y)
{
	make_root(x);
	x->setf(y, -1);
}

inline void cut(Node* x, Node* y)
{
	make_root(y);
	access(x);
	splay(x);
	y->fa = x->ch[0] = nil;
	y->t = -1;
	x->up();
}

inline Node& path(Node* x, Node* y)
{
	make_root(x);
	access(y);
	splay(y);
	return *y;
}

struct Edge {
	int u, v, t;
	bool operator<(const Edge& rhs) const
	{
		return u < rhs.u || (u == rhs.u && v < rhs.v);
	}
} E[MAX_M];
bool b[MAX_M];

bool cmp(int i, int j)
{
	return E[i].t < E[j].t;
}

namespace UF {
	int f[MAX_N + 1], rk[MAX_N + 1];
	int F(int x)
	{
		return f[x] ? f[x] = F(f[x]) : x;
	}

	inline void merge(int x, int y)
	{
		if (rk[x] < rk[y]) swap(x, y);
		f[y] = x;
		rk[x] += rk[x] == rk[y];
	}
}

void build(vector<int> &e)
{
	using UF::F;
	using UF::merge;
	
	sort(e.begin(), e.end(), cmp);
	Rep (i, 0, e.size()) {
		int u = E[e[i]].u, v = E[e[i]].v, ru = F(u), rv = F(v);
		if (ru != rv) {
			merge(ru, rv);
			Node* z = nodes + n + 1 + e[i];
			z->v = E[e[i]].t;
			link(z, nodes + u);
			link(z, nodes + v);
		}
	}
}

struct Event {
	int k, u, v, id;
} Q[MAX_Q];

inline int ID(const Edge& e)
{
	return lower_bound(E, E+m, e) - E;
}

namespace io {
	const int MAX_B = 2.7e7;
	char buff[MAX_B], * st = buff, * ed;
	inline void read()
	{
		ed = st + fread(buff, sizeof(char), MAX_B, stdin);
	}

	template<typename T>
	inline void scan(T& x)
	{
		char c;
		while (c = *st++, !isdigit(c)) ;
		x = c - '0';
		while (c = *st++, isdigit(c)) x = c - '0' + x*10;
	}
};

using io::scan;

int main()
{
	io::read();
	int q;
	scan(n), scan(m), scan(q);
	Rep (i, 0, m) {
		scan(E[i].u), scan(E[i].v), scan(E[i].t);
		if (E[i].u > E[i].v) swap(E[i].u, E[i].v);
	}
	sort(E, E+m);
	
	Rep (i, 0, q) {
		scan(Q[i].k), scan(Q[i].u), scan(Q[i].v);
		if (Q[i].k == 2) {
			if (Q[i].u > Q[i].v) swap(Q[i].u, Q[i].v);
			Q[i].id = ID((Edge){Q[i].u, Q[i].v});
			b[Q[i].id] = true;
		}
	}
	
	vector<int> e, ans;
	Rep (i, 0, m)
		if (!b[i])
			e.push_back(i);
	build(e);
	
	Down (i, q-1, 0) {
		Node* x = nodes + Q[i].u, * y = nodes + Q[i].v;
		if (Q[i].k == 1)
			ans.push_back(path(x, y).mx->v);
		else {
			Node* z = nodes + n + Q[i].id + 1, * w = path(x, y).mx;
			z->v = E[Q[i].id].t;
			if (w->v > z->v) {
				const Edge& e = E[w - (nodes + 1 + n)];
				cut(w, nodes + e.u);
				cut(w, nodes + e.v);
				link(z, x);
				link(z, y);
			}
		}
	}

	Down (i, ans.size()-1, 0)
		printf("%d\n", ans[i]);
	return 0;
}
```