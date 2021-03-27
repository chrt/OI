---
title: "[APIO 2016] Fireworks"
date: 2017-05-06 22:38:49
categories:
- APIO
tags:
- 凸壳
- 可并堆
---
给一棵n个内部结点, m个叶子的有根树, 边带权ci. 修改边权, 使所有叶子到根的距离相等, 求最小代价. 将c修改为c' (c'&ge;0), 代价为abs(c-c'). (1&le;n+m&le;3\*10^5, 1&le;ci&le;10^9)
Link: WMD 神犇在 APIO 2016 时讲题的课件
<!--more-->
一道好题, 不过一知半解......

- subtask 1: n=1, 1&le;m&le;100. 将所有边权修改为它们的中位数.
- subtask 2: 1&le;n+m&le;300, 且根到任一叶子的距离不超过300. 内部结点到它后代中的叶子的距离也必须相等, 所以可以自底向上地DP: 设dp[i][j]表示使子树i中所有叶子到i的距离等于j的最小代价, dp'[i][j]表示子树i中所有叶子到fa[i]的距离等于j的最小代价, 则:
```
dp[x][j] = sum { dp'[y][j] | y is son of x }
dp'[x][j] = min { abs(w-c[x]) + dp[x][j-w] | 0 <= w <= j }
```

也许应该对距离 "离散化" ?

接下来的优化很妙.

y = f(x) = |x-x[1]| + |x-x[2]| + ... + |x-x[k]|. 这个函数的图象是两条射线和一些线段形成的下凸壳. 设 H(x) = 使所有叶子到根的距离等于x的最小代价. 当 n=1 时, H 拥有 f 的形式.

事实上, 无论n取多少, H都是由两条射线和一些线段形成的下凸壳, 虽然我不会证明.

类似于DP, 对每棵子树u, 维护凸壳H(u, x), H'(u, x). 设H(u,x)的最值在[L,R]处取得, c[u] = w, 则:
```
H(u, x) = sum { H'(v, x) | v is son of u }
H'(u, x) = {
	  H(u, x) + w			x < L
	  H(u, L) - x + L + w	L <= x < L+w
	  H(u, L) 	  	  		L+w <= x < R+w
	  H(u, L) + x - R - w	x >= R+w
}
```

怎样理解H'(u, x)的递推式呢? 把添加父边后的调整看作三步:
1. 添加一条长度为w的父边
2. 固定父边的长度不变, 减短/增长子树内的边
3. 增长/减短父边, 加上修改的代价

H(u,x)在[L,R]处是一个点, 或者水平的一段, 两边斜率的绝对值均不小于1. 因此, 在两边的时候, 应尽量修改父边, 避免修改子树内的边; 同时, 注意边权不能为负数, 所以增长是无限的, 而至多缩短w.

发现H->H'只需将(-inf, L)向上平移w, 用两条斜率为-1,0的线段, 一条斜率为1的射线替换[L, inf).

下凸壳的特点是斜率单增. H的每一段都是直的, 并且斜率为自然数. 令每个转折点处斜率+1, 如果要使斜率在此处增加更多, 只需在该处多放几个转折点.

1. H'最右一段的斜率总是1, 所以H(u, x)最右一段的斜率等于u的儿子的数目.
2. H(u, 0) = 子树u中所有边权之和.

由以上两个事实, 发现只用记录转折点的横坐标. 每一段可以用点斜式直接以解析的方式表达.

H'->H, 仅需要将所有H'的转折点合并. 尽管我不会证明, 但是画一画图可以发现这是正确的.

所以, 我们用可并堆维护凸壳.

DFS一遍, 即可自底向上地计算出H(root,x). 然后, 算出每一段的解析式, 用端点更新答案.

```cpp
typedef long long ll;

const int N = 3e5;

struct Node {
	ll v;
	Node* lc, * rc;
} node[2*N], * root[N+1], * ptr = node;

Node* merge(Node* x, Node* y)
{
	if (!x) return y;
	if (!y) return x;
	if (y->v > x->v) swap(x, y);
	x->lc = merge(x->lc, y);
	swap(x->lc, x->rc);
	return x;
}

inline void pop(Node* &x)
{
	x = merge(x->lc, x->rc);
}

inline void push(Node* &x, ll v)
{
	ptr->v = v;
	x = merge(x, ptr++);
}

struct Edge {
	int to, w;
};

int n, m;
vector<Edge> adj[N+1];

void dfs(int u, int w)
{
	Node*& r = root[u];
	rep (i, 0, adj[u].size()) {
		int v = adj[u][i].to, c = adj[u][i].w;
		if (v <= n) {
			dfs(v, c);
			r = merge(r, root[v]);
		} else {
			push(r, c);
			push(r, c);
		}
	}

	if (u == 1) return;
	
	ll L, R;
	per (i, adj[u].size(), 2)
		pop(r);
	R = r->v;
	pop(r);
	L = r->v;
	pop(r);

	push(r, L+w);
	push(r, R+w);
}

ll x[2*N];

int main()
{
	ll x0 = 0, y0 = 0; // y = k(x-x0) + y0
	
	scanf("%d%d", &n, &m);
	rep (i, 2, n+m+1) {
		int p, c;
		scanf("%d%d", &p, &c);
		adj[p].push_back((Edge){i, c});
		y0 += c;
	}
	
	dfs(1, 0);

	Node*& r = root[1];
	int top = 0;
	while (r) {
		x[top++] = r->v;
		pop(r);
	}

	int k = (int)adj[1].size() - top;
	ll ans = y0;
	while (top--) {
		ll t = x[top];
		y0 += k++ * (t-x0);
		x0 = t;
		ans = min(ans, y0);
		while (top && x[top-1] == t) ++k, --top;
	}

	printf("%lld\n", ans);
	return 0;
}
```