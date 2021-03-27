---
title: "[bzoj 3694] 最短路"
date: 2017-06-05 14:32:33
categories:
- bzoj
tags:
- LCA
- STL
---
给出一个n个点m条边的无向图 (边带权) 和该图的一棵最短路树, 对于除源点1外的每个点i, 求1到i的最短路, 要求不经过i在最短路树上的父边. (n&le;4000, m&le;10^5, 1&le;边权&le;10^5)
<!--more-->
假设点 $u$ 存在一条符合要求的路径, 考虑在子树 $u$ 外的最后一个点 $v$ 和在子树 $u$ 内的第一个点$w$ (它们必然是存在的). $v$ 和 $w$ 由一条非树边连接. 根据最短路树的性质, $1-v$ 和 $w-u$ 都沿着树边走.

量化一下. 设 $1$ 到 $u$ 的距离为 $l(u)$. 目标是最小化 $l(v)+l(v,w)+l(w)-l(u)$, 即最小化 $l(v)+w(v,w)+l(w)$, 其中 $w\in subtree(u), v\not\in subtree(u)$. 我们来维护对当前点 $u$ 有贡献的非树边. 先合并它的儿子的非树边集合, 然后删掉两端点均在子树 $u$ 内的那些边, 最后遍历一遍邻接表, 加入和 $u$ 关联, 且另一端在子树 $u$ 外的非树边. 我们怎么知道集合中哪些边的两端点均在子树内呢? 虽然点数少, 但是边数多, 所以暴力检查是不可行的. 但是每条边只会加入一次, 再删除一次. $(v,w)$ 将在 $lca(v,w)$ 处删除. 集合用`multiset`就好, 合并可以启发式.

看了下题解......直接轻重链剖分+线段树就可以了: $(v,w)$ 对 $v-lca(v,w)-w$ 路径上除 $lca(v,w)$ 的点有贡献. 我在想怎么合并, 没从每条边的贡献这个角度思考. [WZH学长的题解](http://blog.csdn.net/qantun_mechanics/article/details/52649998). 吐槽一下, 找LCA为什么要用倍增......QwQ 好吧学长说他是为了练习板子......QwQ

轻重链剖分+线段树的时间复杂度是 $O(n+m\lg^2 n)$. 我的算法的时间复杂度......因为有删除, 并且不是按照集合的大小来合并的, 所以通常的启发式合并的时间复杂度的分析方法不适用. 考虑轻重链剖分. 和每个点关联的边集只会在经过该点到根的路径上的轻边时合并到另一个集合中. 每条边只会关联两个点. 因此总的时间复杂度是 $O(n+m\lg m\lg n)$. 怪不得慢一些.

据说本题数据是随机生成的, 树高是期望 $O(\lg n)$ 的......

```cpp
typedef multiset<int> Set;

const int N = 4001;

struct Edge
{
	int to, w;
};
vector<int> L[N];
vector<Edge> G[N], T[N];

int dfn, st[N], ed[N], fa[N], d[N], l[N], sz[N], mx[N], top[N], ans[N];
Set S[N];

void dfs_1(int u)
{
	sz[u] = 1;
	rep (i, 0, T[u].size())
	{
		int v = T[u][i].to;
		if (v == fa[u]) continue;
		d[v] = d[u] + 1;
		l[v] = l[u] + T[u][i].w;
		fa[v] = u;
		dfs_1(v);
		sz[u] += sz[v];
		if (sz[v] > sz[mx[u]]) mx[u] = v;
	}
}

void dfs_2(int u, int t)
{
	st[u] = ++dfn;
	top[u] = t;
	if (mx[u])
	{
		dfs_2(mx[u], t);
		rep (i, 0, T[u].size())
		{
			int v = T[u][i].to;
			if (v != fa[u] && v != mx[u]) dfs_2(v, v);
		}
	}
	ed[u] = ++dfn;
}

inline int lca(int x, int y)
{
	while (top[x] != top[y])
	{
		if (d[top[x]] > d[top[y]]) swap(x, y);
		y = fa[top[y]];
	}
	return d[x] > d[y] ? y : x;
}

void dfs(int u, Set& s)
{
	rep (i, 0, T[u].size())
	{
		int v = T[u][i].to;
		if (v != fa[u]) dfs(v, v == mx[u] ? s : S[v]);
	}
	rep (i, 0, T[u].size())
	{
		int v = T[u][i].to;
		if (v != fa[u] && v != mx[u])
		{
			s.insert(S[v].begin(), S[v].end());
			S[v].clear();
		}
	}
	rep (i, 0, L[u].size())
	{
		Set::iterator it = s.find(L[u][i]);
		s.erase(it);
	}
	rep (i, 0, G[u].size())
	{
		int v = G[u][i].to, t = l[v] + G[u][i].w + l[u];
		if (st[v] >= st[u] && ed[v] <= ed[u]) continue;
		s.insert(t);
		L[lca(v, u)].push_back(t);
	}
	ans[u] = s.empty() ? -1 : *s.begin() - l[u];
}

int main()
{
	int n, m;
	scanf("%d%d", &n, &m);
	rep (i, 0, m)
	{
		int a, b, l, t;
		scanf("%d%d%d%d", &a, &b, &l, &t);
		if (a == b) continue;
		vector<Edge> *adj = t ? T : G;
		adj[a].push_back((Edge){b, l});
		adj[b].push_back((Edge){a, l});
	}
	dfs_1(1);
	dfs_2(1, 1);
	dfs(1, S[1]);
	rep (i, 2, n+1)
		printf("%d ", ans[i]);
	puts("");
	return 0;
}
```