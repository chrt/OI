---
title: "[bzoj 3224] Tyvj 1728 普通平衡树"
date: 2017-02-13 12:27:41
categories:
- 模板
tags:
- 平衡树
- Treap
---
Treap。由于有重复元素，需记录出现次数。
<!--more-->
```cpp
{% raw %}
#include <cstdio>
#include <cstdlib>

const int MAX_N = 1e5 + 1;

struct Node {
	Node* ch[2];
	int v, r, s, t;
	void up()
	{
		s = ch[0]->s + ch[1]->s + t;
	}
} nodes[MAX_N], * null = nodes, * root = null;

namespace Treap {
	Node* new_node(int v)
	{
		static int p = 1;
		nodes[p] = (Node){{null, null}, v, rand(), 1, 1};
		return nodes + p++;
	}

	void rotate(Node* &x, int d)
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
			++x->t, ++x->s;
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
			if (x->t > 1)
				--x->t, --x->s;
			else {
				if (x->ch[0] == null)
					x = x->ch[1];
				else if (x->ch[1] == null)
					x = x->ch[0];
				else {
					int d = x->ch[1]->r > x->ch[0]->r;
					rotate(x, d);
					remove(x->ch[d^1], v);
					x->up();
				}
			}
		} else {
			remove(x->ch[v > x->v], v);
			x->up();
		}
	}

	Node* kth(Node* x, int k)
	{
		while (x != null) {
			int s = x->ch[0]->s, t = x->t;
			if (s < k && s+t >= k)
				return x;
			if (k <= s)
				x = x->ch[0];
			else {
				k -= s + t;
				x = x->ch[1];
			}
		}
		return null;
	}

	int rank(Node* x, int v)
	{
		int r = 0;
		while (x->v != v) {
			if (v < x->v)
				x = x->ch[0];
			else {
				r += x->ch[0]->s + x->t;
				x = x->ch[1];
			}
		}
		return r + x->ch[0]->s + 1;
	}

	Node* pre(Node* x, int v)
	{
		Node* y = null;
		while (x != null) {
			if (v <= x->v)
				x = x->ch[0];
			else {
				y = x;
				x = x->ch[1];
			}
		}
		return y;
	}

	Node* suc(Node* x, int v)
	{
		Node* y = null;
		while (x != null) {
			if (v >= x->v)
				x = x->ch[1];
			else {
				y = x;
				x = x->ch[0];
			}
		}
		return y;
	}
}

int main()
{
	int n;
	scanf("%d", &n);
	while (n--) {
		int opt, x;
		scanf("%d %d", &opt, &x);
		using namespace Treap;
		switch (opt) {
			case 1:
				insert(root, x);
				break;
			case 2:
				remove(root, x);
				break;
			case 3:
				printf("%d\n", rank(root, x));
				break;
			case 4:
				printf("%d\n", kth(root, x)->v);
				break;
			case 5:
				printf("%d\n", pre(root, x)->v);
				break;
			case 6:
				printf("%d\n", suc(root, x)->v);
		}
	}
	return 0;
}
{% endraw %}
```