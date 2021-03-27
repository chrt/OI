---
title: "[bzoj 4025] 二分图"
date: 2017-06-07 12:37:21
categories:
- bzoj
tags:
- 分治
- 并查集
- LCT
- 二分图
---
n个节点的无向图, 总时间为T. m条在一个时间段内存在的边, 求每一时刻该图是否是二分图. (n&le;10^5, m&le;2\*10^5, T&le;10^5)
<!--more-->
一个图是二分图当且仅当不存在奇环.

树是二分图.

我们也可以这样判断: 任取原图的一棵生成树. 如果所有非树边的两端点在树上的距离是奇数, 则原图是二分图; 否则不是.

---
**并查集 + 对时间分治**
如果只有加边操作, 带权并查集就可以搞定. 维护每个点到它在并查集中的根, 在原图中某棵生成树上的距离的奇偶性. AB的奇偶性 xor BC的奇偶性 = AC的奇偶性.

并查集怎样支持撤销呢? 我们可以用一个栈记录所有的修改, 回到原来的状态.

用线段树分解每条边存在的时间区间. 这样, 任意两个区间的关系只有包含, 相离两种. 遍历这棵线段树即可.

并查集不用路径压缩. 我也不知道具体原理, 据说是因为相关复杂度分析在这里失效.

时间复杂度 $O((m+T)\lg T\lg n)$.

贴一下 [Educational Codeforces Round 22](/2017/06/07/cf813/) F题的代码:
```cpp
// c++11 required

const int N = 1e5 + 1;

struct UF
{
	int f[N], rk[N], d[N], top;
	pair<int*, int> S[3*N];

	int F(int x, int& t)
	{
		return f[x] ? (t ^= d[x], F(f[x], t)) : x;
	}

	void merge(int x, int y, int t)
	{
		if (rk[x] > rk[y]) swap(x, y);
		S[top++] = make_pair(f+x, f[x]);
		f[x] = y;
		S[top++] = make_pair(d+x, d[x]);
		d[x] = t;
		if (rk[x] == rk[y])
			S[top++] = make_pair(rk+y, rk[y]++);
	}

	void retrace(int t)
	{
		while (top != t)
		{
			pair<int*, int>& o = S[--top];
			*o.first = o.second;
		}
	}
} U;

struct Edge {
	int x, y, L, R;
};

bool ans[N];

void solve(int l, int r, vector<Edge>& E)
{
	if (E.empty()) return;
	
	vector<Edge> a, b;
	int tmp = U.top, m = (l+r)/2;
	
	for (auto e : E)
	{
		if (e.L <= l && r <= e.R)
		{		
			int dx = 0, dy = 0, x = U.F(e.x, dx), y = U.F(e.y, dy);
			if (x == y)
			{
				if (dx == dy)
				{
					rep (i, l, r+1) ans[i] = false;
					U.retrace(tmp);
					return;
				}
			}
			else
			{
				U.merge(x, y, dx ^ dy ^ 1);
			}
		}
		else
		{
			if (e.L <= m) a.push_back(e);
			if (e.R > m) b.push_back(e);
		}
	}

	solve(l, m, a);
	solve(m+1, r, b);

	U.retrace(tmp);
}

map<pair<int, int>, int> S;
vector<Edge> E;

int main()
{
	int n, q;
	scanf("%d%d", &n, &q);
	rep (i, 0, q)
	{
		pair<int, int> e;
		scanf("%d%d", &e.first, &e.second);
		if (e.first > e.second) swap(e.first, e.second);
		if (S.count(e))
		{
			E.push_back((Edge){e.first, e.second, S[e], i-1});
			S.erase(e);
		}
		else
		{
			S[e] = i;
		}
	}
	for (auto t : S)
		E.push_back((Edge){t.first.first, t.first.second, t.second, q-1});
	rep (i, 0, q) ans[i] = true;
	solve(0, q-1, E);
	rep (i, 0, q) puts(ans[i] ? "YES" : "NO");
	return 0;
}
```

---
**LCT**
这种方法我还怎么理解......

维护以删除时间为边权的最大生成树. 非树边如果连接两个距离为偶数的点, 则加入一个集合中. 当且仅当集合为空, 原图是二分图.

关于删除时间的最大生成树, 保证了先删非树边, 再删树边. 删除一条树边, 不会引起非树边向树边的切换.

疑惑: 一条非树边是否加入集合, 我们考察的是它是否在一个仅有一条非树边的奇环中. 这样是否全面? 维护最大生成树的操作, 可能导致一些点对间距离奇偶性的切换, 而我们并没有显式地考虑这些变化. 这个集合的确切定义是什么?

如果阅读本文的您清楚这种解法的原理, 恳请告知. QwQ