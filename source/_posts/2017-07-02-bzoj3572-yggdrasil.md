---
title: "[bzoj 3572] [Hnoi2014]世界树"
date: 2017-07-02 08:54:01
categories:
- bzoj
tags:
- 虚树
- 树形DP
- 倍增
---
一棵边权均为1的无根树, q个询问: 给出m个关键点, 每个点隶属于与它最近的关键点, 距离相同取编号最小的, 求每个关键点管多少点. (n,q,Σm &le; 3\*10^5)
<!--more-->
[虚树学习笔记 | Sengxian's Blog](https://blog.sengxian.com/algorithms/virtual-tree)

虚树是一棵包含所有关键点和关键点两两之间LCA的树, 祖先-后代关系和原树保持一样.

建虚树的过程简述如下:
1. 加入根节点. 把关键点按原树中前序遍历的DFS序排列. 去重.
2. 模拟 DFS 的过程, S 即递归栈, 元素为当前点和它的祖先. 根节点入栈.
3. 设栈顶元素为 u (由于加入了根节点, S 始终非空), 新加入点 v. 设 w = lca(u,v). 如果 w = u, 说明 v 是 u 的后代, 直接压入栈中.
4. 否则, w 包含 u 的这一棵子树访问完毕. 从栈顶第二个点 S[top-2] 向 S[top-1] 连边, 直到栈中只剩下一个点, 或 dep[S[top-2]] < dep[w]. 那么, S[top-1] 是深度 &ge; dep[w] 的最后一个点 (对于栈中只剩一个点的情况, 由于根节点没有祖先, 它显然是最后一个). 如果 S[top-1] ≠ w, 则从 w 向 S[top-1] 连边, 并且用 w 替换 S[top-1] 成为栈顶. 压入点 v.
5. 最后, 弹栈, 连边, 直到 S 为空.

根节点加不加其实无所谓啦~ 但是DP的时候也会方便一点.

那这道题怎么做呢? 虚树上除了询问点, 还有它们之间的LCA. 每条边上还可能挂着一堆子树......如果能知道每个询问点在虚树上管哪一部分就好办了.

学习了一下题解. 虚树上每个点被谁管, 相比每个询问点管谁, 更加容易处理. 两遍DP, 维护最近次近 (要求不在同一棵子树中; 把上方的所有点也看成一棵子树) 即可.

对于虚树上的一条边 (u,v) (不含端点), 设 belong[u] = x, belong[v] = y. 如果 x = y, 那么 (u,v) 也属于 x. 否则, 先判断 u-x, v-y 相差太悬殊的情形. 那么接下来, (u,v) 上存在一个分界点 z, 使得 (z,u) 属于 x, (v,z] 属于 y. 列个不等式可以解出它的位置. 如果解出一个整数, 那么该点与 x,y 距离相等, 属于哪边取决于 x,y 的编号大小. 用倍增找到这个点, 累加答案.

对于虚树上的一个点 u, 把它自己和挂在点 u 上且不在虚树中的点的数目累加给 belong[u]. 不用考虑*上方的部分*, 因为我们保证根在虚树中.

```cpp
typedef pair<int, int> ii;

const int N = 3e5 + 1, D = 20, inf = (1<<30)-1;

bool q[N];
int n, dfs_clock, max_d, num, dfn[N], anc[N][D], fa[N], up[N], dep[N], sz[N], V[N], ans[N];
vector<int> adj[N], T[N];

void dfs(int u, int p)
{
	dfn[u] = ++dfs_clock;
	anc[u][0] = p;
	dep[u] = dep[p] + 1;
	sz[u] = 1;
	rep (i, 0, max_d)
		anc[u][i+1] = anc[anc[u][i]][i];
	rep (i, 0, adj[u].size())
	{
		int v = adj[u][i];
		if (v != p)
		{
			dfs(v, u);
			sz[u] += sz[v];
		}
	}
}

int jump(int x, int h)
{
	for (int i = 0; h; ++i, h >>= 1)
		if (h & 1)
			x = anc[x][i];
	return x;
}

int lca(int x, int y)
{
	if (dep[y] < dep[x]) swap(x, y);
	y = jump(y, dep[y]-dep[x]);
	if (x == y) return y;
	for (int i = max_d; i >= 0; --i)
		if (anc[x][i] != anc[y][i])
			x = anc[x][i], y = anc[y][i];
	return anc[x][0];
}

bool cmp(int i, int j)
{
	return dfn[i] < dfn[j];
}

struct VirtualTree
{
	int t, cnt, S[N], tmp[N];

	inline void link(int x, int y)
	{
		fa[y] = x;
		T[x].push_back(y);
		tmp[cnt++] = y;
	}

	void build()
	{
		sort(V, V+num, cmp);
		t = 0, cnt = 0;
		S[t++] = V[0];
		rep (i, 1, num)
		{
			int p = lca(S[t-1], V[i]);
			if (S[t-1] != p)
			{
				while (t > 1 && dep[S[t-2]] >= dep[p])
				{
					link(S[t-2], S[t-1]);
					--t;
				}
				if (S[t-1] != p)
				{
					link(p, S[t-1]);
					S[t-1] = p;
				}
			}
			S[t++] = V[i];
		}
		while (t > 1)
		{
			link(S[t-2], S[t-1]);
			--t;
		}
		tmp[cnt++] = S[0];
		fa[S[0]] = 0;
		copy(tmp, tmp + cnt, V);
		num = cnt;
	}
} VT;

struct Info
{
	ii x[2];
	Info()
	{
		x[0] = x[1] = ii(inf, 0);
	}
	void operator|=(const ii& t)
	{
		if (t <= x[0])
			x[1] = x[0], x[0] = t;
		else if (t < x[1])
			x[1] = t;
	}
} f[N];

void dp()
{
	rep (i, 0, num)
	{
		int u = V[i];
		f[u] = Info();
		if (q[u]) f[u] |= ii(0, u);
		rep (j, 0, T[u].size())
		{
			int v = T[u][j];
			f[u] |= ii(f[v].x[0].first + dep[v] - dep[u], f[v].x[0].second);
		}
	}
	per (i, num-1, 0)
	{
		int u = V[i], p = fa[u];
		ii t = f[p].x[f[p].x[0].second == f[u].x[0].second];
		f[u] |= ii(t.first + dep[u] - dep[p], t.second);
	}
}

inline int calc(int x, int y)
{
	return sz[x] - sz[y];
}

void solve()
{
	rep (i, 0, num-1)
	{
		int v = V[i], u = fa[v], l = dep[v]-dep[u], w = jump(v, l-1),
			x = f[u].x[0].second, y = f[v].x[0].second;
		up[v] = w;
		if (l == 1) continue;
		if (x == y)
		{
			ans[x] += calc(w, v);
		}
		else
		{
			int a = f[u].x[0].first, b = f[v].x[0].first;
			if (a-b <= -l)
			{
				ans[x] += calc(w, v);
			}
			else if (a-b >= l)
			{
				ans[y] += calc(w, v);
			}
			else
			{
				int t = (a-b+l)/2 - (!((a-b+l) & 1) && x < y), z = jump(v, t);
				ans[x] += calc(w, z);
				ans[y] += calc(z, v);
			}
		}
	}
	rep (i, 0, num)
	{
		int u = V[i], s = sz[u];
		rep (i, 0, T[u].size())
			s -= sz[up[T[u][i]]];
		ans[f[u].x[0].second] += s;
	}
}

void clear()
{
	rep (i, 0, num)
	{
		int v = V[i];
		T[v].clear();
		q[v] = false;
		ans[v] = 0;
	}
}

int main()
{
	scanf("%d", &n);
	rep (i, 0, n-1)
	{
		int x, y;
		scanf("%d%d", &x, &y);
		adj[x].push_back(y);
		adj[y].push_back(x);
	}
	while ((1<<max_d) < n) ++max_d;
	dfs(1, 0);
	int m;
	scanf("%d", &m);
	while (m--)
	{
		static int k, h[N];
		scanf("%d", &k);
		num = k;
		rep (i, 0, k)
		{
			scanf("%d", h+i);
			V[i] = h[i];
			q[h[i]] = true;
		}
		if (!q[1]) V[num++] = 1;
		VT.build();
		dp();
		solve();
		rep (i, 0, k)
			printf("%d ", ans[h[i]]);
		puts("");
		clear();
	}
	return 0;	
}
```