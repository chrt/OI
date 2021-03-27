---
title: "[bzoj 2555] SubString"
date: 2017-05-11 22:13:47
categories:
- bzoj
tags:
- 后缀自动机
- LCT
---
给一个字符串, 支持以下操作, 强制在线:
- 在当前字符串的后面插入一个字符串
- 询问字符串s在当前字符串中出现了多少次

(字符集为大写英文字母, 字符串最终长度&le;6\*10^5, 询问次数&le;10^4, 询问总长度&le;3\*10^6)
<!--more-->
如果没有后端插入操作, 做法见[[spoj NSUBSTR] Substrings](/2017/04/21/spoj-nsubstr/).

现在有后端插入操作, 需要支持Parent树的形态的修改和子树求和, 不妨用LCT来维护. 子树求和既可以转化为链修改, 又可以使用[[spoj QTREE 6 7] Query on a tree VI VII](/2017/04/28/spoj-qtree67/)中所述的*LCT维护子树信息*的方法. 我采用了前者.

实现的时候注意, SAM分裂结点时, 子树求和的结果也要一并复制. 由于使用了延迟标记, 得 push down 一遍才能得到被复制的结点的真实值.

```cpp
const int N = 6e5, M = 3e6, SIGMA = 26;

struct Node {
	Node* trans[SIGMA], * link, * fa, * ch[2];
	int len, t, a, v;
	
	Node();
	
	void add(int d)
	{
		a += d;
		v += d;
	}
	
	void down()
	{
		if (a) {
			ch[0]->add(a);
			ch[1]->add(a);
			a = 0;
		}
	}

	void setf(Node* f, int _t)
	{
		t = _t;
		fa = f;
		t >= 0 ? f->ch[t] = this : 0;
	}

	void clone(Node*);
	
} node[2*N], null, * nil = &null, * root = node;

int ptr = 1;

Node::Node(): link(0), fa(0), len(0), t(-1), a(0), v(0)
{
	fill_n(trans, SIGMA, (Node*)0);
	ch[0] = ch[1] = nil;
}

inline void rot(Node* y)
{
	Node* x = y->fa;
	int t = y->t;
	y->setf(x->fa, x->t);
	y->ch[t^1]->setf(x, t);
	x->setf(y, t^1);
}

void splay(Node* x)
{
	static Node* S[2*N], * y;
	int top = 0;
	
	for (y = x; y->t >= 0; y = y->fa) S[top++] = y;
	for (y->down(); top; S[--top]->down()) ;

	while (x->t >= 0) {
		if ((y = x->fa)->t >= 0)
			rot(x->t ^ y->t ? x : y);
		rot(x);
	}
}

void Node::clone(Node* o)
{
	splay(o);
	copy(o->trans, o->trans + SIGMA, trans);
	link = o->link;
	v = o->v;
}

void access(Node* x)
{
	for (Node* y = nil; x; y = x, x = x->fa) {
		splay(x);
		x->ch[1]->t = -1;
		y->setf(x, 1);
	}
}

inline void link(Node* x, Node* y)
{
	splay(y);
	y->setf(x, -1);
}

inline void cut(Node* x)
{
	access(x);
	splay(x);
	x->ch[0]->setf(0, -1);
	x->ch[0] = nil;
}

void add(Node* x)
{
	access(x);
	splay(x);
	x->add(1);
}

void extend(int c)
{
	static Node* last = root;
	Node* p = last, * now = node + ptr++;
	now->len = p->len + 1;
	last = now;	
	
	while (p && !p->trans[c]) {
		p->trans[c] = now;
		p = p->link;
	}

	if (!p) {
		now->link = root;
	} else {
		Node* q = p->trans[c];
		if (q->len == p->len + 1) {
			now->link = q;
		} else {
			Node* nq = node + ptr++;
			nq->clone(q);
			nq->len = p->len + 1;
			now->link = q->link = nq;
		
			cut(q);
			link(nq->link, nq);
			link(nq, q);

			while (p && p->trans[c] == q) {
				p->trans[c] = nq;
				p = p->link;
			}
		}
	}

	link(now->link, now);
	add(now);
}

int query(char* s)
{
	Node* x = root;
	for (; *s; ++s) {
		x = x->trans[*s - 'A'];
		if (!x) return 0;
	}
	splay(x);
	return x->v;
}

void decode_with_mask(char s[], int n, int mask)
{
	rep (i, 0, n) {
		mask = (mask * 131 + i) % n;
		swap(s[i], s[mask]);
	}
}

char s[N+1];

int main()
{
	int q, mask = 0;
	scanf("%d%s", &q, s);
	for (int i = 0; s[i]; ++i)
		extend(s[i] - 'A');
	while (q--) {
		char type[6];
		static char str[M+1];
		
		scanf("%s%s", type, str);
		int n = strlen(str);
		decode_with_mask(str, n, mask);
		
		if (type[0] == 'Q') {
			int ans = query(str);
			printf("%d\n", ans);
			mask ^= ans;
		} else {
			rep (i, 0, n)
				extend(str[i] - 'A');
		}
	}
	return 0;
}
```

