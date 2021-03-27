---
title: "[APIO 2015] Jakarta Skyscrapers"
date: 2017-05-02 10:02:24
categories:
- APIO
tags:
- 最短路
- 根号算法
---
n座摩天楼, n只doge. 每只doge有初始位置B[i]和跳跃能力P[i], 收到消息后, 位于摩天楼b的i号doge可以:
- 跳到b-P[i], b+P[i] (如果不越界)
- 把消息传递给当前摩天楼上的其他doge

求0号doge将消息传给1号doge所需的最少跳跃总步数. (0≤Bi&lt;n, 1≤n≤30000, 1≤Pi≤30000, 2≤m≤30000)
<!--more-->

---
1年前, W学长提议一起做往年的APIO, 每星期一套.

第一个星期, 点开APIO 2012. 啥都不会做. 学长说这道Kunai的暴力好像不难写, 我们来写暴力吧, 我表示滋磁. 然后一个下午过去了, 我5分, 超越学长5分. QAQ

第二个星期, 点开APIO 2015. 发现本题会暴力建图跑最短路! 不想看其他题了, 学一学正解. 这是什么鬼啊?? 为什么会扯到分块?? 学长身体不适, 鼓励我自行研究题解, 然后在机房拉了一张床睡觉.

第三个星期, 我们不想从事这个活动了.

照了题解写了写, 但是没过UOJ上的hack数据. 昨天试着在先前的程序上修改, 今天直接重写, 终于AC了. 一道充满回忆的题.

![screenshot](/images/apio2015-skyscraper-screenshot.png)

---
首先, 暴力建图: B[i]向(B[i] + k P[i])连长度为|k|的边. 这样忽略了doge传完消息后停在某摩天楼的事实, 但不影响最优解.

然后, 发现: p很大时, 连不了几条边; p很小时, 对于同一个p, 这样处理 (以p=2为例):
![construction](/images/apio2015-skyscraper.jpg)

设阈值为s, 图的点数为sn, 边数为(mn/s+3sn). 令s=(m/3)^0.5.

然后跑一发单源最短路即可. 两个优化:
- 不用把图显式地用邻接表建出来. 跑最短路时直接判断, 减小空间和常数.
- 进一步改造图的结构, 使所有边权为1, 这样可以用BFS代替单源最短路算法.

```cpp
const int N = 3e4, M = 3e4, S = 100, inf = (1<<30)-1;

struct Node {
	int r, c, d;
	bool operator>(const Node& o) const
	{
		return d > o.d;
	}
};

vector<int> L[2][N];
int n, m, B[M], P[M], d[S][N];
priority_queue<Node, vector<Node>, greater<Node> > Q;

inline void relax(int r, int c, int _d)
{
	if (d[r][c] > _d) {
		d[r][c] = _d;
		Q.push((Node){r, c, _d});
	}
}

int Dijkstra()
{
	fill_n(*d, sizeof(d)/sizeof(int), inf);
	Q.push((Node){0, B[0], 0});
	d[0][B[0]] = 0;

	while (!Q.empty()) {
		Node x = Q.top();
		Q.pop();

		int r = x.r, u = x.c;
		if (x.d > d[r][u]) continue;
		if (u == B[1]) return x.d;
		
		if (r) {
			relax(0, u, x.d);
			if (u+r < n) relax(r, u+r, x.d + 1);
			if (u >= r) relax(r, u-r, x.d + 1);
		} else {
			rep (i, 0, L[0][u].size())
				relax(L[0][u][i], u, x.d);
			rep (i, 0, L[1][u].size()) {
				int l = L[1][u][i];
				for (int j = 1, k = u+l; k < n; ++j, k += l)
					relax(0, k, x.d + j);
				for (int j = 1, k = u-l; k >= 0; ++j, k -= l)
					relax(0, k, x.d + j);
			}
		}
	}
	return -1;
}

int main()
{
	scanf("%d%d", &n, &m);
	int s = sqrt(m/3)+0.5;
	rep (i, 0, m) {
		scanf("%d%d", B+i, P+i);
		L[P[i] >= s][B[i]].push_back(P[i]);
	}
	printf("%d\n", Dijkstra());
	return 0;
}
```