---
title: "[bzoj 3261] 最大异或和: 可持久化Trie"
date: 2017-03-01 22:37:43
categories:
- 模板
tags:
- 可持久化
- Trie
- 贪心
---
给定一个初始长度为N的非负整数序列和M个操作: 在序列末尾添加一个数x / 寻找l&le;p&le;r, 使得A[p] xor A[p+1] xor ... xor A[N] xor x最大. (N,M&le;300000, 0&le;a[i]&le;10^7)
<!--more-->
1. 给定一个集合S和一个数x, 怎样寻找S的元素y, 使得x xor y最大?
给S集合中的位串建立一棵Trie树, 从高位往低位贪心.

2. 给定一个序列{A}和一些询问: 求y属于A[l, r], 使得x xor y最大.
给序列{A}的前缀建立可持久化Trie树, 则A[l, r]是第r棵, 第(l-1)棵Trie树i之差.

3. 给定一个序列{A}和一些询问: 求p属于[l, r], 使得A[p] xor A[p+1] xor ... xor A[N] xor x最大. 这正是本题的询问.
异或运算满足结合律. 以下将xor记为^. 设x^y=z, 则x^(x^y)=(x^x)^y=y=x^z, 因此, 一个区间的异或和=两个前缀区间的异或和 相异或. 记S[i] = A[1]^A[2]^...^A[N], S[0] = 0, 则A[p]^A[p+1]^...^A[N]^x = S[p-1]^S[N]^x = S[p-1]^(S[N]^x), 问题转化为, 寻找i属于[l-1, r-1], 使得S[i]^(S[N]^x)最大. 这就是问题2.

Tip: 这里有两个意义上的前缀和: 前缀异或和, 单串Trie树的前缀和. 前者是后者维护的量. S[0] = 0是一个有意义的量, 所以空的Trie树需放在-1位置, 或者特判. 关于-1下标, 在Po姐的博客里看到一种[有趣的写法](http://blog.csdn.net/popoqqq/article/details/48656499).

```cpp
#include <bits/stdc++.h>
#define Rep(i, a, b) for (int i = (a), i##_ = (b); i < i##_; ++i)
#define For(i, a, b) for (int i = (a), i##_ = (b); i <= i##_; ++i)
#define Down(i, a, b) for (int i = (a), i##_ = (b); i >= i##_; --i)
#define bin(x, k) ((x)>>(k)&1)
using namespace std;
const int D = 24, MAX_N = 3e5, MAX_M = 3e5;

struct Node {
	Node* ch[2];
	int v;
} nodes[(D+1)*(MAX_N+MAX_M+1)+1], * nil = nodes, * root[MAX_N + MAX_M + 1];

inline Node* new_node(const Node& x)
{
	static Node* cur = nodes + 1;
	*cur = x;
	return cur++;
}

void insert(Node* &o, Node* p, int x, int k)
{
	o = new_node(*p);
	++o->v;
	if (k < 0) return;
	int b = bin(x, k);
	insert(o->ch[b], p->ch[b], x, k-1);
}

int query(Node* o, Node* p, int x, int k)
{
	if (k < 0) return 0;
	int b = !bin(x, k);
	return o->ch[b] - p->ch[b] ? 1 << k | query(o->ch[b], p->ch[b], x, k-1)
		: query(o->ch[!b], p->ch[!b], x, k-1);
}

inline void init()
{
	*nil = (Node){nil, nil, 0};
}

int main()
{
	int n, m, S = 0;
	scanf("%d%d", &n, &m);
	init();
	insert(root[0], nil, 0, D-1);
	For (i, 1, n) {
		int a;
		scanf("%d", &a);
		insert(root[i], root[i-1], S ^= a, D-1);
	}
	while (m--) {
		char c;
		int l, r, x;
		scanf(" %c", &c);
		if (c == 'A') {
			scanf("%d", &x);
			insert(root[n+1], root[n], S ^= x, D-1);
			++n;
		} else {
			scanf("%d%d%d", &l, &r, &x);
			printf("%d\n", query(root[r-1], l > 1 ? root[l-2] : nil, x^S, D-1));
		}
	}
	return 0;
}
```