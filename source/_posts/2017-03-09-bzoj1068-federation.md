---
title: "[bzoj 1068] [SCOI2005]王室联邦"
date: 2017-03-09 21:38:40
categories:
- bzoj
tags:
- 块状树
- 分块
- 构造
---
把一棵N个点的无根树分成一些块, 使得每块的大小属于[B, 3B], 并且每块存在一个点, 块内所有点到该点的路径上的点均在块内 (但是该点可以在块外). 构造一种方案 (输出块数, 每个点所属的块的编号, 每块对应的那个特殊点, 允许多对一), 或者报告无解. (1&le;N&le;1000, 1&le;B&le;N)
<!--more-->
我觉得做题有三个方面的乐趣: yy算法, 码码码 & debug, 对比其他解法 & 交流. 看了题解第一方面的乐趣就会少很多......所幸本题保有这份乐趣.

只有平凡情况 (B&gt;N) 才会无解吗? 3B这个上限有什么奥妙? 特殊点的限制意味着什么?

先解决第三个疑问. 特殊点的限制意味着, 要么块中所有的点在一个连通块内, 要么块分解出的连通块均与块外某点相邻 (此时该点即为特殊点).

尝试在BFS或DFS时划分每一块. BFS是不行的, 因为从一层到下一层的时候多不满足特殊点限制. DFS见一个点塞一个也是不行的, 将导致底部某些零碎的点无法进入任何一块. 考虑自底向上地构造, 对子树DFS, 把子树中未能在本树完成分块的点集合起来分配, 再向上返回这次剩下的点. 可以像Tarjan算法一样, 用一个栈记录访问过的点.

[B, 3B]的范围比较宽. 先考虑每不少于B个立即分到一块中. 这样, 块的大小属于[B, 2B). 子树中剩下的点加上当前点不会超过B个, 任意合并均可满足限制. 3B的道理便在此处.

在树根处也可能剩下不超过B个点, 将它们与某个块合并即可. 为了确保在非平凡情况下能找到可以合并的块, 所有点DFS剩下的那不超过B个点都不急于合并, 而是留在栈内, 并返回这些点的具体个数给它的父亲. 这样, 除了根剩下的点, 其余块的大小均属于[B, 2B), 任取一块连通的即可.

怎么找那连通的一个块, 我犯了蠢, 又去DFS一番......事实上, 合并到先前最后生成的那一块即可. 由此可以看出, 只有B&gt;N的平凡情况才会无解.

或许是使用STL的缘故, 和许多0ms相比跑得有些慢 (20ms+).

```cpp
#include <bits/stdc++.h>
#define Rep(i, a, b) for (int i = (a), i##_ = (b); i < i##_; ++i)
#define For(i, a, b) for (int i = (a), i##_ = (b); i <= i##_; ++i)
#define Down(i, a, b) for (int i = (a), i##_ = (b); i >= i##_; --i)

using namespace std;
const int N = 1e3;

int n, k, b, belong[N+1], center[N+1];
vector<int> adj[N+1];
stack<int> S;

int dfs(int u, int p)
{
	int sum = 0;
	Rep (i, 0, adj[u].size()) {
		int v = adj[u][i];
		if (v == p) continue;
		sum += dfs(v, u);
		if (sum >= b) {
			center[++k] = u;
			Rep (j, 0, sum) {
				int w = S.top();
				S.pop();
				belong[w] = k;
			}
			sum = 0;
		}
	}
	S.push(u);
	return sum + 1;
}

int main()
{
	scanf("%d%d", &n, &b);
	if (b > n) {
		puts("0");
		return 0;
	}
	Rep (i, 1, n) {
		int u, v;
		scanf("%d%d", &u, &v);
		adj[u].push_back(v);
		adj[v].push_back(u);
	}

	Rep (i, 0, dfs(1, 0)) {
		belong[S.top()] = k;
		S.pop();
	}

	printf("%d\n", k);
	For (i, 1, n)
		printf("%d%c", belong[i], " \n"[i == n]);
	For (i, 1, k)
		printf("%d%c", center[i], " \n"[i == k]);
	return 0;
}
```