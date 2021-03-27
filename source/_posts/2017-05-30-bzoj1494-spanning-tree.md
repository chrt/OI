---
title: "[bzoj 1494] [NOI2007]生成树计数"
date: 2017-05-30 14:41:47
categories:
- bzoj
tags:
- 生成树
- 动态规划
- 矩阵乘法
---
n个点标号1~n, u,v间有一条无向边当且仅当0&lt;|u-v|&le;k, 求该图生成树的数目模65521. (k&le;5, n&le;10^15)
<!--more-->
看到数据范围, 大概是矩阵乘法优化状压DP.

---
题面中提到了**矩阵树定理**: 无向图G=(V,E)的生成树数目 = 基尔霍夫矩阵 (度数矩阵-邻接矩阵) 的任一(|V|-1)阶余子式. G可以有重边, 自环.

暂时没有学习这个定理的证明.

**行列式**的求法: 高斯消元.
- 交换两行, 行列式取反.
- `row(i) += k row(j) (i != j)`, 行列式不变.
- 三角矩阵的行列式等于主对角线元素之积.

会写暴力咯~

猜测: 暴力打个表, 再上BM算法, 可以得到比状压DP更短的递推式. 看到过有人这么做, 求递推式用的是Matlab. 但是这两个目前都不会. TAT

---
从前往后考虑: 点i和i-k~i-1中的哪些连边.

设$f(i,S)$为前i个点, 最后k个的连通性 (最小表示) 为S, 前(i-k)个都和最后k个点中的某一个连通, 这样的子图的生成树的数目.

如果某点v&le;i-k不和后k个点中的任何一个连通, 那么, 它就再也不能和它们连通了, 因为v和编号大于i的点没有边相连. 反之, 我们可以保证, 如果最后k个点在一个连通块内, 前(i-k)个点也在这个连通块内.

这个限制类似于, 轮廓线DP求解用1\*2的骨牌铺满n\*m的网格的方案数, 要求轮廓线以上的格子全部铺满.

先枚举出有标号的k个点 (边数不超过10) 的所有生成森林, 以此给S编号, 并求出DP的初值. 然后, 枚举新加入一个点和这k个点中的哪些连边, 求出所有的转移. 为了满足限制, 如果即将退出S的那个点在一个大小为1的连通块中, 那么, 它必须和新加入的点连边.

状态数不超过52.

提交的代码就不放了, 因为我把表打到程序里了. 以下是打表的程序 (C++11好评; 感觉这份代码写得非常......随性?):
```cpp
#define x first
#define y second

using namespace std;

typedef basic_string<int> State;
typedef pair<int, int> Edge;

const int K = 5;

int k, f[K], trans[1000][1000];
map<State, int> s, id;
vector<Edge> e;

int F(int x)
{
	return f[x] == -1 ? x : f[x] = F(f[x]);
}

inline void reset()
{
	fill_n(f, k, -1);
}

int main()
{
	scanf("%d", &k);
	rep (i, 1, k+1) rep (j, 0, k-i) e.push_back(Edge(j, j+i));

	rep (i, 0, 1<<e.size()) {
		reset();
		bool ok = true;
		rep (j, 0, e.size()) if (i & (1<<j)) {
			int x = F(e[j].x), y = F(e[j].y);
			if (x == y) { ok = false; break; }
			f[y] = x;
		}
		if (!ok) continue;
		int tmp[K];
		fill_n(tmp, k, -1);
		State t;
		
		rep (j, 0, k) {
			int x = F(j);
			if (tmp[x] == -1) tmp[x] = j;
			t += tmp[x];
		}
		++s[t];
	}

	int cnt = 0;
	for (auto t : s)
		id[t.first] = cnt++;
	
	for (auto t : id) {
		State fr = t.first, to;

		int y = k-1;
		rep (j, 1, k) if (fr[j] == 0) { y = j-1; break; }
		
		rep (i, 0, 1<<k) {
			if (y == k-1 && !(i & 1)) continue;
			bool b[K], ok = true;
			fill_n(b, K, false);
			rep (j, 0, k) if (i & (1<<j)) {
				if (b[fr[j]]) { ok = false; break; }
				b[fr[j]] = true;
			}
			if (!ok) continue;
			int x = k-1;
			rep (j, 1, k) if (b[fr[j]]) { x = j-1; break; }

			to = fr.substr(1) + x;
			rep (j, 1, k) {
				if (b[fr[j]]) to[j-1] = x;
				else if (fr[j] == 0) to[j-1] = y;
				else --to[j-1];
			}
			assert(id.count(to));
			++trans[t.second][id[to]];
		}
	}
	
	int ed;
	printf("%d\n\n{", ed = (int)id.size());
	int now = 0;
	for (auto t : s) {
		printf("%d%c", t.second, ",}"[now == ed-1]), ++now;
	}
	puts("\n");
	rep (i, 0, id.size()) {
		putchar('{');
		rep (j, 0, id.size()) printf("%d%c", trans[i][j], ",}"[j == (int)id.size()-1]);
		if (i != (int)id.size()-1) putchar(',');
		puts("");
	}
	return 0;
}
```