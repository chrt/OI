---
title: "[bzoj 2329] [HNOI2011]括号修复"
date: 2017-04-14 21:06:53
categories:
- bzoj
tags:
- Splay
- 贪心
---
维护一个长度为N的由`(`和`)`组成的字符串, 支持四类操作:
- Replace a b c: 将[a,b]之间的所有字符改为c. (c=`(` / `)`)
- Swap a b: 将[a,b]之间的字符串翻转.
- Invert a b: 将[a,b]之间的左右括号互换.
- Query a b: 询问[a,b]之间的字符串至少要改变多少位才能变成合法 (可以匹配) 的括号序列, 保证有解.

(N,M&le;10^5)
<!--more-->
感觉和最大连续子段和很像, 应该要找到某种递推关系, 在Splay等数据结构上打打标记维护.

但是我只会$O(n^3)$的暴力DP......猜想某种贪心是可行的, 但还是找不到递推式.

题解说可以把所有匹配的括号去掉, 然后就变成了`)))(((`这种形式. (虽然不清楚这样贪心的原理)

可是然后呢......

再看题解, 令`)=-1`, `(=1`, 则`)`的数目等于最小和前缀, `(`的数目等于最大和后缀 (空串合法且值为0). 去掉所有匹配的括号后, 这显然正确. 每加上一对`()`, 这个性质仍然保持. 还真转化成了最大连续子段和问题.

考虑修改. 单独看三种操作, 都好搞. 只需要维护
```
value: v, sum, sz, min_pre, max_pre, min_suf, max_suf
tag: cover, rev, inv
```

考虑三种标记之间的联系. 先看`cover`和`rev & inv`.

打上赋值标记, 其他标记可直接清除. 打翻转标记, 如果已有赋值标记, 则忽略翻转标记. 打反转标记, 如果已有赋值标记, 则重新赋值.

因此, `cover`和`rev & inv`不共存.

再看`rev`和`inv`. 如果同时有这两种标记, 它们的先后次序是无所谓的. 我是将它们写成矩阵之后看到这一点的......也可以yy一番.

但是答案怎么求呢? 设`)`的数目是`a`, `(`的数目是`b`, 那么答案=`ceil(a/2) + ceil(b/2)`. 即
```
))))(((((( -> ()()()()()
)))((((( -> ()()()()
```

问题又来了, 怎么证明这就是最优解呢?

......

```cpp
const int N = 1e5;

struct Node {
	Node* ch[2], * fa;
	int t;
	int v, pre_min, pre_max, suf_min, suf_max, sum, sz, c;
	short tag;
	
	Node(int=0, int=0);
	void* operator new(size_t);
	void setf(Node*, int);
	void replace(int);

	void reverse()
	{
		if (c) return;
		tag ^= 1;
		swap(ch[0], ch[1]);
		ch[0]->t = 0;
		ch[1]->t = 1;
		swap(pre_min, suf_min);
		swap(pre_max, suf_max);
	}

	void inverse()
	{
		if (c) replace(c == 1 ? -1 : 1);
		else {
			tag ^= 2;
			pre_min = -pre_min;
			pre_max = -pre_max;
			suf_min = -suf_min;
			suf_max = -suf_max;
			sum = -sum;
			v = -v;
			swap(pre_min, pre_max);
			swap(suf_min, suf_max);
		}
	}

	void down()
	{
		if (c) {
			ch[0]->replace(c);
			ch[1]->replace(c);
			c = 0;
		} else {
			if (tag & 1) {
				ch[0]->reverse();
				ch[1]->reverse();
			}
			if (tag & 2) {
				ch[0]->inverse();
				ch[1]->inverse();
			}
			tag = 0;
		}
	}

	void up()
	{
		pre_min = min(ch[0]->pre_min, ch[0]->sum + v + ch[1]->pre_min);
		pre_max = max(ch[0]->pre_max, ch[0]->sum + v + ch[1]->pre_max);
		suf_min = min(ch[1]->suf_min, ch[1]->sum + v + ch[0]->suf_min);
		suf_max = max(ch[1]->suf_max, ch[1]->sum + v + ch[0]->suf_max);
		sum = ch[0]->sum + v + ch[1]->sum;
		sz = ch[0]->sz + 1 + ch[1]->sz;
	}
} nodes[N+3], * nil = nodes, * root = nil;

Node::Node(int _v, int s): fa(0), t(-1), sz(s), c(0), tag(0)
{
	ch[0] = ch[1] = nil;
	sum = v = _v;
	pre_min = suf_min = min(v, 0);
	pre_max = suf_max = max(v, 0);
}

void* Node::operator new(size_t)
{
	static Node* ptr = nodes + 1;
	return ptr++;
}

inline void Node::setf(Node* f, int _t)
{
	t = _t;
	fa = f;
	t >= 0 ? f->ch[t] = this : root = this;
}

inline void Node::replace(int _v)
{
	if (this == nil) return;
	tag = 0;
	c = v = _v;
	if (v == 1) {
		suf_min = pre_min = 0;
		sum = suf_max = pre_max = sz;
	} else {
		sum = suf_min = pre_min = -sz;
		suf_max = pre_max = 0;
	}
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

void splay(Node* x, Node* r=0)
{
	Node* y;
	while (x->fa != r) {
		if ((y = x->fa)->fa != r)
			rot(x->t ^ y->t ? x : y);
		rot(x);
	}
	x->up();
}

Node* get_kth(int k, Node* r=0)
{
	Node* x = root;
	while (true) {
		x->down();
		if (x->ch[0]->sz + 1 == k) return splay(x, r), x;
		if (x->ch[0]->sz >= k) x = x->ch[0];
		else k -= x->ch[0]->sz + 1, x = x->ch[1];
	}
}

Node* get_segment(int l, int r)
{
	get_kth(l);
	Node* x = get_kth(r+2, root);
	return x->ch[0];
}

char s[N+2];

Node* build(int l, int r)
{
	if (l > r) return nil;
	int m = (l+r)/2;
	Node* x = new Node(s[m] == '(' ? 1 : -1, 1);
	if (l == r) return x;
	build(l, m-1)->setf(x, 0);
	build(m+1, r)->setf(x, 1);
	x->up();
	return x;
}

int main()
{
	int n, m;
	scanf("%d%d%s", &n, &m, s+1);
	root = build(0, n+1);
	while (m--) {
		int a, b;
		char c;
		scanf("%s%d%d", s, &a, &b);
		Node* x = get_segment(a, b);
		switch (s[0]) {
		case 'R':
			scanf(" %c", &c);
			x->replace(c == '(' ? 1 : -1);
			x->fa->up();
			root->up();
			break;
		case 'S':
			x->reverse();
			x->fa->up();
			root->up();
			break;
		case 'I':
			x->inverse();
			x->fa->up();
			root->up();
			break;
		default:
			a = -x->pre_min;
			b = x->suf_max;
			printf("%d\n", (a+1)/2 + (b+1)/2);
		}
	}
	return 0;
}
```