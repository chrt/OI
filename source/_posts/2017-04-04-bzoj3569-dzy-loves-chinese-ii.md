---
title: "[bzoj 3569] DZY Loves Chinese II"
date: 2017-04-04 15:08:52
categories:
- bzoj
tags:
- 线性基
- 高斯消元
- 随机化
---
n个点m条边无向连通图, q个询问: 删除指定的k条边, 图是否连通 (询问后立即复原). 强制在线. (n&le;10^5, m&le;5\*10^5, 5\*10^4, 1&le;k&le;15)
建议感受一下原题面, 并感受一下 [bzoj 3563] DZY Loves Chinese.
<!--more-->
不会捉啊......

题解是这样说的. 搞一棵DFS生成树, 给后向边随机赋值, 令树边的权值等于所有覆盖它的后向边权值的异或和. 删除k条边后图不连通 <=> 这k条边的权值存在一个异或和等于0的非空子集.

不少题解说: "图不连通当且仅当删除一条树边和覆盖这条树边的所有边集", 这显然不对......有的题解还提了一下只删非树边的情形, 也不够......

![bzoj3569](/images/bzoj3569.jpg)

如图, 删除(2,3), (1,3), (3,4), (3,5)可使图不连通, 它们的权值满足异或和为0, 但不属于上述两种情况.

感觉运行在一般图上的算法通常证明起来不容易......

假设我们画了画图, 接受了这个算法的正确性 (虽然有一定概率出错), 接下来考虑程序怎么写. 判定是否存在异或和等于0的子集高斯消元可以搞定. 赋值的过程可以通过两遍DFS实现. 第一遍, 给后向边赋值, 并标记后向边所覆盖的链. 第二遍, 根据结点上的标记, 自底向上地给树边赋值.

做法挺神的. 遗憾之处在于, 不知道作者是怎样想到这种奇妙构造的......如果知道了, 也许就能证明正确性了.

```cpp
#define rand() (rand()%(RAND_MAX-1) + 1)

const int N = 1e5, K = 15, M = 5e5;

struct Edge {
	int k, v;
};

vector<Edge> adj[N + 1];
int mark[N + 1], fa[N + 1], v[K], val[M];
bool vis[N + 1], pass[M];

inline void add(int k, int u, int v)
{
	adj[u].push_back((Edge){k, v});
}

bool check(int k)
{
	int now = 0;
	for (int x = 1<<30; now < k && x; x >>= 1) {
		int r = now;
		while (r < k && !(v[r] & x)) ++r;
		if (r == k) continue;
		swap(v[now], v[r]);
		Rep (i, now+1, k) if (v[i] & x) v[i] ^= v[now];
		++now;
	}
	return now == k;
}

void dfs(int u)
{
	vis[u] = true;
	Rep (i, 0, adj[u].size()) {
		Edge& e = adj[u][i];
		if (pass[e.k]) continue;
		pass[e.k] = true;
		if (vis[e.v]) {
			int r = rand();
			val[e.k] = r;
			mark[e.v] ^= r;
			mark[u] ^= r;
		} else {
			fa[e.v] = u;
			dfs(e.v);
		}
	}
}

int dfs_2(int u)
{
	int s = mark[u];
	Rep (i, 0, adj[u].size()) {
		Edge& e = adj[u][i];
		if (fa[e.v] == u) {
			int r = dfs_2(e.v);
			s ^= r;
			val[e.k] ^= r;
		}
	}
	return s;
}

int main()
{
	srand(0x542b);
	
	int n, m;
	scanf("%d%d", &n, &m);
	Rep (i, 0, m) {
		int x, y;
		scanf("%d%d", &x, &y);
		add(i, x, y);
		add(i, y, x);
	}
	dfs(1);
	dfs_2(1);

	int key = 0, q;
	scanf("%d", &q);
	Rep (i, 0, q) {
		int k;
		scanf("%d", &k);
		Rep (j, 0, k) {
			int c;
			scanf("%d", &c);
			--(c ^= key);
			v[j] = val[c];
		}
		if (check(k)) {
			++key;
			puts("Connected");
		} else
			puts("Disconnected");
	}
	return 0;
}
```