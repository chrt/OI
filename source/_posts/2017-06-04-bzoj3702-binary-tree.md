---
title: "[bzoj 3702] 二叉树"
date: 2017-06-04 09:45:03
categories:
- bzoj
tags:
- 线段树
- 逆序对
---
给一棵所有非叶子节点都有两个儿子的二叉树. 每个叶子节点上有一个权值, 它们是1..n的一个排列. 可以任意交换每个非叶子节点的左右孩子. 求所有叶子节点的权值按中序遍历写出来, 逆序数的最小值. (2&le;n&le;2\*10^5)
<!--more-->
注意到每个非叶子节点对答案的贡献是固定的, 与两棵子树内的交换无关. 因此,
$$
ans = \sum_{\text{非叶子节点}v} \min\left\\{f(v), s(lc(v)) s(rc(v)) - f(v)\right\\}\\\\
s(v) = \text{子树 }v\text{ 中叶子节点的个数}\\\\
f(u) = \sum_{v\in lc(u)}\sum_{w\in rc(u)} [c_v < c_w]
$$

求 $f$ 我写了个启发式合并+线段树合并, $O(n\lg^2 n)$ 卡着时限过......数组开小, RE两发.

事实上, 可以在线段树合并的过程中统计 $f$ - $[l,m]$ 对 $[m+1,r]$ 的贡献, 和CDQ分治是一样的. 时间复杂度 $O(n\lg n)$.

该算法提供了一个思路: 挖掘数据结构本身的性质, 顺便统计一些量, 而不是仅仅把它们当作模块组合起来.

代码如下:

```cpp
typedef long long ll;

const int N = 2e5 + 1, NLGN = 19*N;

struct Seg {
	int lc[NLGN], rc[NLGN], v[NLGN], ptr;
	
	void build(int p, int& o, int l, int r)
	{
		o = ++ptr;
		if (l == r) return v[o] = 1, void();
		int m = (l+r)/2;
		if (p <= m) build(p, lc[o], l, m);
		else build(p, rc[o], m+1, r);
		v[o] = v[lc[o]] + v[rc[o]];
	}

	ll merge(int& x, int y, int l, int r)
	{
		if (!x || !y) return x += y, 0;
		ll sum = 1LL * v[lc[x]] * v[rc[y]];
		if (l != r) {
			int m = (l+r)/2;
			sum += merge(lc[x], lc[y], l, m) + merge(rc[x], rc[y], m+1, r);
		}
		v[x] += v[y];
		return sum;
	}
} S;

int n, dfn, rt[2*N];
ll ans;

struct Tree {
	int lc[2*N], rc[2*N], c[2*N], sz[2*N], ptr;

	void read()
	{
		int t;
		read(t);
	}
	
	void read(int& i)
	{
		i = ++ptr;
		scanf("%d", c+i);
		if (!c[i])
			read(lc[i]), read(rc[i]);
	}

	void dfs(int o)
	{
		if (c[o]) return S.build(c[o], rt[o], 1, n), sz[o] = 1, void();
		int x = lc[o], y = rc[o];
		dfs(x);
		dfs(y);
		sz[o] = sz[x] + sz[y];
		ll sum = S.merge(rt[o] = rt[x], rt[y], 1, n);
		ans += min(sum, 1LL * sz[x] * sz[y] - sum);
	}
} T;

int main()
{
	scanf("%d", &n);
	T.read();
	T.dfs(1);
	printf("%lld\n", ans);
	return 0;
}
```