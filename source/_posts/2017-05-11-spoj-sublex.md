---
title: "[spoj SUBLEX] Lexicographical Substring Search"
date: 2017-05-11 14:54:00
categories:
- spoj
tags:
- 后缀自动机
---
求S去重后字典序第K小的子串, Q个询问, 保证合法. (S&le;9\*10^4, Q&le;500, 0&lt;K&lt;2^31)
<!--more-->
建出后缀自动机, 递推求出DAG上从每个点出发的路径数, 并计算某点走前i条边路径数 (前缀和). 查询和二叉搜索树找第K大类似.

DFS不要忘了记忆化.

确定走哪条边从前往后暴力找就可以了, 二分反而会超时......

```cpp
#include <bits/stdc++.h>
#define rep(i, a, b) for (int i = (a), i##_end = (b); i < i##_end; ++i)
#define per(i, a, b) for (int i = (a), i##_end = (b); i >= i##_end; --i)

using namespace std;

typedef unsigned uint;

const int N = 9e4, SIGMA = 26;

struct Node {
	Node* ch[SIGMA], * fa;
	int len;
	uint sum[SIGMA];
} node[2*N], * root = node;

int ptr = 1;

void add(int c)
{
	static Node* last = root;
	Node* now = node + ptr++, * p = last;
	last = now;
	now->len = p->len + 1;

	while (p && !p->ch[c]) {
		p->ch[c] = now;
		p = p->fa;
	}

	if (!p) {
		now->fa = root;
		return;
	}

	Node* q = p->ch[c];
	if (q->len == p->len + 1) {
		now->fa = q;
	} else {
		Node* nq = node + ptr++;
		*nq = *q;
		nq->len = p->len + 1;
		now->fa = q->fa = nq;
		while (p && p->ch[c] == q) {
			p->ch[c] = nq;
			p = p->fa;
		}
	}
}

bool vis[2*N];

uint dfs(Node* u)
{
	bool& v = vis[u-node];
	if (v) return u->sum[SIGMA-1];
	v = true;
	
	uint s = 1;
	rep (i, 0, SIGMA) {
		Node* v = u->ch[i];
		if (v) s += dfs(v);
		u->sum[i] = s;
	}
	return s;
}

void query(uint k)
{
	Node* x = root;
	++k;
	while (k > 1) {
		int c = 0;
		while (x->sum[c] < k) ++c;
		putchar('a' + c);
		k -= c ? x->sum[c-1] : 1;
		x = x->ch[c];
	}
	putchar('\n');
}

char s[N+1];

int main()
{
	scanf("%s", s);
	int n = 0;
	while (s[n])
		add(s[n++] - 'a');
	dfs(root);
	int q;
	scanf("%d", &q);
	while (q--) {
		uint k;
		scanf("%u", &k);
		query(k);
	}
	return 0;
}
```