---
title: "[bzoj 4871] [Shoi2017]摧毁“树状图”"
date: 2017-05-29 14:20:22
categories:
- bzoj
tags:
- 动态规划
- 树
---
删掉树上两条边不相交的路径 (可退化为点), 问最多裂成多少个连通块. (多组数据, ∑ n ≤ 5 × 10^5)
<!--more-->
这当然是一道树型DP, 但是我好晕啊......又去学习了一下WA神犇的题解, 发现自己方向不错, 但是少了一步转化. WA神犇的思路十分清晰, 分类讨论恰到好处. 本来挺腌臢的一道题......顿时变得平易近人, 甚至有种美感?

记 $deg_T(v)$ 为 $T$ 中 $v$ 的度数. 从树 $T$ 中删掉一个点 $v$, 得到 $deg_T(v)$ 个连通块; 删掉两个点 $u,v$, 得到 $deg_T(u)+deg_T(v)-2$ 个连通块; 删掉一棵树 $T'\subset T$, 得到
$$
\begin{array} {}
D(T')
&= \sum_{v\in T'} (deg_T(v) - deg_{T'}(v)) \\
&= \sum_{v\in T'} deg_T(v) - 2|E(T')| \\
&= \sum_{v\in T'} deg_T(v) - 2(|V(T')|-1) \\
&= \sum_{v\in T'} (deg_T(v)-2) + 2
\end{array}
$$
个连通块. (我缺少这步转化, 没有抽象出"度数之和", 从而难以简化问题)

记 $d(v) = deg_T(v)-2$.

现在, 删掉两条边不相交的链 $P,H$. 根据它们的位置关系, 分三种情况讨论:
- 有一个公共点. $P\cup H$ 是从该点出发的不超过4条链. 得到 $D(P\cup H)$ 个连通块.
- 点导出子图有一条公共边. 即, 存在 $(u,v)\in E(T)$, 使得 $u\in V(P), v\in V(H)$. 得到 $D(P\cup H)$ 个连通块.
- 其他. 即, 存在一个点 $v \not\in P\cup H$, 将 $T$ 分割成两部分 $T_1,T_2$, 使得 $P\subset T_1, H\subset T_2$. 得到 $D(P)+D(H)-1=\sum_{v\in P\cup H} d(v) + 3$ 个连通块.

现在问题变得很简单了~

任取一个根.

$f(u)$: 从u向下的所有链, $\sum_v d(v)$ 的最大值, $g(u)$: 所有以u为lca的路径, $\sum_v d(v)$ 的最大值. 这两个值可以一遍DFS求出.

记 $sum(S,k)$ 为从 $\left\\{f(v)|v\in S\right\\}$ 中取不超过 $k$ 个数, 和的最大值 (定义取0个数和为0). $adj(u)$ 为 $T$ 中所有和 $u$ 相邻的点的集合.

- 有一个公共点. 设该点为 $u$, 用 $sum(adj(u),4) + d(u) + 2$ 更新答案.
- 点导出子图有一条公共边. 设该边为 $(u, v), u = fa(v)$, 则 $v$ 是其中一条路径的lca. 用 $sum(adj(u)-\left\\{v\right\\}, 2) + g(v) + 2$ 更新答案.
- 其他. 设 $P$ 的lca为 $x$, $H$ 的lca为 $y$, 分为两种情况:
  - $x$ 和 $y$ 有祖先-后代关系 (不妨设$x$是$y$的祖先), 设 $x$ 到 $y$ 的路径上, 除 $x$ 以外的第一个点是 $z$; 
  - 没有祖先-后代关系, 设 $w=lca(x,y)$.
设 $h(u)$ 为 $u$ 的子树 (包含 $u$) 中 $g$ 的最大值, $h'(u)$ 为 $u$ 的子树 (不包含 $u$) 中 $g$ 的最大值.
分别用 $sum(adj(x)-\left\\{z\right\\}, 2) + h'(z) + 3$ 和 $sum(adj(w)-\left\\{fa(w)\right\\}, 2) + 3$ 更新答案.

可以用一个`struct`维护 $f(adj(u))$ 的前4大. 要从集合中排除一个元素, 和我们维护的最大值, 次大值做做比较即可.

至此, 问题解决, 撒花~

```cpp
#define Z(x) max(0, x)

const int N = 5e5, inf = 1e9;

template<typename T>
inline void upmax(T& x, T v)
{
	x = max(x, v);
}

struct Info {
	int v[4];
	Info(int t=-inf)
	{
		v[0] = v[1] = v[2] = v[3] = t;
	}
	void operator+=(int t)
	{
		rep (i, 0, 4) if (t >= v[i]) swap(t, v[i]);
	}
	int one(int t=-inf)
	{
		return max(0, t == v[0] ? v[1] : v[0]);
	}
	int two(int t=-inf)
	{
		if (t == v[0]) return Z(v[1]) + Z(v[2]);
		if (t == v[1]) return Z(v[0]) + Z(v[2]);
		return Z(v[0]) + Z(v[1]);
	}
	int four()
	{
		return Z(v[0]) + Z(v[1]) + Z(v[2]) + Z(v[3]);
	}
} f[N];

int ans, F[N], g[N], h[N], d[N];
vector<int> adj[N];

int dfs_1(int u, int p)
{
	f[u] = -inf;
	rep (i, 0, adj[u].size()) {
		int v = adj[u][i];
		if (v != p) {
			f[u] += dfs_1(v, u);
		}
	}
	g[u] = f[u].two() + d[u];
	return F[u] = f[u].one() + d[u];
}

void dfs_2(int u, int p, int t)
{
	f[u] += t;
	h[u] = t = -inf;
	rep (i, 0, adj[u].size()) {
		int v = adj[u][i];
		if (v != p) {
			dfs_2(v, u, f[u].one(F[v]) + d[u]);
			upmax(ans, f[u].two(F[v]) + d[u] + max(g[v] + 2, h[v] + 3));
			upmax(h[v], g[v]);
			upmax(h[u], h[v]);
			upmax(ans, h[v] + t + 3);
			upmax(t, h[v]);
		}
	}
	upmax(ans, f[u].four() + d[u] + 2);
}

int main()
{
	int T, x, n;
	scanf("%d%d", &T, &x);
	x *= 2;
	while (T--) {
		scanf("%d", &n);
		rep (i, 0, x) scanf("%*d");
		rep (i, 0, n) adj[i].clear();
		rep (i, 0, n-1) {
			int u, v;
			scanf("%d%d", &u, &v);
			--u, --v;
			adj[u].push_back(v);
			adj[v].push_back(u);
		}
		rep (i, 0, n) d[i] = (int)adj[i].size() - 2;
		ans = 0;
		dfs_1(0, -1);
		dfs_2(0, -1, -inf);
		printf("%d\n", ans);
	}
	return 0;
}
```