---
title: "[51nod 1766] 树上最远点对"
date: 2017-04-12 18:16:16
categories:
- 51nod
tags:
- 树
- 线段树
- LCA
---
给一棵n个结点边带权的无根树, m个询问, 求标号分别在[a,b], [c,d]中的两点间的最远距离. (n&le;10^5, m&le;10^5, 1&le;权值&le;10^4)
<!--more-->
[树的直径](/2017/04/12/trees-diameter/)

怎样从无到有发现一个神奇的性质呢?

```cpp
const int N = 1e5;

struct Edge {
	int v, w;
};
vector<Edge> adj[N+1];
int n;

namespace Distance {
	const int D = 17;
	int top = 0, dep[2*N], num[N+1], st[2*N][D+1], Log[2*N];

	void dfs(int u, int p)
	{
		num[u] = top;
		st[top++][0] = dep[u];
		rep (i, 0, adj[u].size()) {
			Edge& e = adj[u][i];
			if (e.v != p) {
				dep[e.v] = dep[u] + e.w;
				dfs(e.v, u);
				st[top++][0] = dep[u];
			}
		}
	}

	void init()
	{
		dfs(1, 0);
		rep (i, 2, top + 1)
			Log[i] = Log[i >> 1] + 1;
		rep (i, 1, Log[top] + 1) {
			int k = 1<<(i-1);
			rep (j, 0, top-k) {
				st[j][i] = min(st[j][i-1], st[k][i-1]);
				++k;
			}
		}
	}

	inline int query(int u, int v)
	{
		int d = dep[u] + dep[v];
		u = num[u];
		v = num[v];
		if (u > v) swap(u, v);
		int t = Log[v-u+1];
		return d - 2 * min(st[u][t], st[v-(1<<t)+1][t]);
	}
}

struct Diameter {
	int x, y, l;
	Diameter(int x=0, int y=0): x(x), y(y)
	{
		l = x == y ? 0 : Distance::query(x, y);
	}
	bool operator<(const Diameter& o) const
	{
		return l < o.l;
	}
	friend Diameter between(const Diameter& a, const Diameter& b)
	{
		Diameter c(a.x, b.x), d(a.x, b.y), e(a.y, b.x), f(a.y, b.y);
		return max(c, max(d, max(e, f)));
	}
	friend Diameter operator+(const Diameter& a, const Diameter& b)
	{
		return max(a, max(b, between(a, b)));
	}
};

struct SegmentTree {
	Diameter d[4*N];
	void build(int o, int l, int r)
	{
		if (l == r) {
			d[o] = Diameter(l, l);
			return;
		}
		int m = (l+r)/2;
		build(o*2, l, m);
		build(o*2+1, m+1, r);
		d[o] = d[o*2] + d[o*2+1];
	}
	
	Diameter query(int L, int R, int o, int l, int r)
	{
		if (L <= l && r <= R) return d[o];
		int m = (l+r)/2;
		if (R <= m) return query(L, R, o*2, l, m);
		if (L > m) return query(L, R, o*2+1, m+1, r);
		return query(L, R, o*2, l, m) + query(L, R, o*2+1, m+1, r);
	}
} T;

template<typename T>
inline void scan(T& x)
{
	char c;
	while (c = getchar(), !isdigit(c)) ;
	x = c - '0';
	while (c = getchar(), isdigit(c)) x = c - '0' + x*10;
}

int main()
{
	scan(n);
	rep (i, 0, n-1) {
		int x, y, z;
		scan(x), scan(y), scan(z);
		adj[x].push_back((Edge){y, z});
		adj[y].push_back((Edge){x, z});
	}
	Distance::init();
	T.build(1, 1, n);
	int m;
	scan(m);
	while (m--) {
		int a, b, c, d;
		scan(a), scan(b), scan(c), scan(d);
		Diameter x = T.query(a, b, 1, 1, n), y = T.query(c, d, 1, 1, n);
		printf("%d\n", between(x, y).l);
	}
	return 0;
}
```