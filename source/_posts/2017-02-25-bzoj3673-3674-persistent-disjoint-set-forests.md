---
title: "[bzoj 3673/3674] 可持久化并查集 by zky/加强版"
date: 2017-02-25 10:12:39
categories:
- bzoj
tags:
- 可持久化
- 并查集
- 平衡树
---
加强版题意: 写一个并查集, n个集合, m个操作, 除了并, 查, 还支持回到某一历史版本, 强制在线. (0&lt;n,m&le;2*10^5)
<!--more-->
原来以为并查集也有自己持久化的方法, 但是, 如果支持回到某一历史版本, 而不仅仅是访问某一历史版本的信息, 这种自底向上记录父亲的数据结构不好搞.

但是, 如果这些信息可以用一个数组表示, 可以转而持久化这个数组. 可持久化数组可以用可持久化线段树/平衡树实现.

不少题解用的是可持久化线段树, 觉得概念上有点怪......需要维护什么区间信息吗? 所以我决定去学学可持久化Treap然后写写这题.

写完之后发现用不着`merge` `split`, 因为只要一开始建好树, 就没有插入和删除操作. 诶? 好像随机权值也不需要了, 那么Treap也不需要, 直接上静态二叉搜索树......`Think twice, code once`很重要啊, 然而做到不容易.

不用路径压缩, 这个数据范围$O(n+m\lg^2 n)$足矣. 由于`Union`操作要做两次修改, 有一些新建的结点就浪费了. 也许垃圾回收是搞这个事的?

bzoj 3674

```cpp
#include <bits/stdc++.h>
#define Rep(i, a, b) for (int i = (a), i##_ = (b); i < i##_; ++i)
#define For(i, a, b) for (int i = (a), i##_ = (b); i <= i##_; ++i)
#define Down(i, a, b) for (int i = (a), i##_ = (b); i >= i##_; --i)
#define fst first
#define sec second
using namespace std;

const int MAX_M = 2e5, MAX_N = 2e5, MLG_N = 18*MAX_M;

struct Node {
	Node* lc, * rc;
	int s, v, rk;
} nodes[MAX_N + MLG_N*2 + 1], * nil = nodes, * root[MAX_M + 1];

typedef pair<int, int> ii;

inline Node* new_node(const Node& v)
{
	static Node* cur = nodes + 1;
	*cur = v;
	return cur++;
}

Node* modify(Node* T, int k, int v, int rk)
{
	Node* o = new_node(*T);
	if (T->lc->s + 1 == k)
		o->v = v, o->rk = rk;
	else if (T->lc->s >= k)
		o->lc = modify(T->lc, k, v, rk);
	else
		o->rc = modify(T->rc, k - T->lc->s - 1, v, rk);
	return o;
}

ii f(Node* o, int x)
{
	while (o != nil) {
		if (o->lc->s + 1 == x)
			return ii(o->v, o->rk);
		if (o->lc->s >= x)
			o = o->lc;
		else
			x -= o->lc->s + 1, o = o->rc;
	}
	assert(0);
}

ii F(Node* now, int x)
{
	ii v = f(now, x);
	return v.fst ? F(now, v.fst) : ii(x, v.sec);
}

Node* Union(Node* now, int x, int y)
{
	ii rx = F(now, x), ry = F(now, y);
	if (rx.fst != ry.fst) {
		now = modify(now, ry.fst, rx.fst, ry.sec);
		if (rx.sec == ry.sec)
			now = modify(now, rx.fst, 0, rx.sec + 1);		
	}
	return now;
}

inline void init()
{
	*nil = (Node){nil, nil, 0, 0, 0};
}

Node* build(int n)
{
	return n ? new_node((Node){build((n-1)/2), build(n/2), n, 0, 0}) : nil;
}

int main()
{
	int n, m, lastans = 0;
	scanf("%d%d", &n, &m);
	init();
	root[0] = build(n);
	For (i, 1, m) {
		int op, k, a, b;
		scanf("%d", &op);
		switch (op) {
		case 1:
			scanf("%d%d", &a, &b);
			root[i] = Union(root[i-1], a^lastans, b^lastans);
			break;
		case 2:
			scanf("%d", &k);
			root[i] = root[k^lastans];
			break;
		default:
			scanf("%d%d", &a, &b);
			root[i] = root[i-1];
			printf("%d\n", lastans = F(root[i], a^lastans) == F(root[i], b^lastans));
		}
	}
	return 0;
}
```