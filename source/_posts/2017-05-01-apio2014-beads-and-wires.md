---
title: "[APIO 2014] Beads and wires"
date: 2017-05-01 14:29:30
categories:
- APIO
tags:
- 树
- 动态规划
---
n个珠子, 两种操作:
- Append(w, v): 将一个新的珠子w和一个已添加的珠子v用红线连起来
- Insert(w, u, v): w是新珠子, u,v是已添加的由红线连接的珠子; 删掉u,v之间的红线, 用两条蓝线分别连接w和u,v

最终得分为蓝线长度之和. 给出结束局面 (珠子的连接方式, 线的长度, 颜色未知), 求最大可能得分. (1≤n≤2\*10^5)
<!--more-->

求最大可能得分, 即, 把线染成红, 蓝, 使得这种局面可以由游戏规则产生, 同时最大化蓝线的长度之和.

开始以为只要蓝线可以配对, 并且每对有一个公共顶点, 每个顶点只关联一对线, 局面就是合法的. 想用网络流, 但是发现不会建同时满足这俩条件的图: 1. 每个顶点只关联一对线. 2. 每条边仅一次参与匹配. 试一试树上DP, 好像可以搞......但是, 疑虑重重: 1. 一场比赛考两道DP? 2. APIO的题这么简单?

WA了......之前想到的是局面合法的必要非充分条件......

![一个反例](/images/apio2014-beads.jpg)

这个局面是不可能产生的. 规则要求始终只能有一个连通块.

现在, 可以等价地重述规则: 从一个珠子开始, 每次可以:
- 添加一颗新的珠子w, 用红线连上已有珠子v
- 添加两颗新的珠子v,w, 选择已有珠子u, 用蓝线连接(u,v), (v,w)

第二个规则也就是说, 以初始的珠子为根, 每次同时添加的两条蓝线不能拐弯.

然后我想到一个部分分算法: 枚举根, 跑最大权匹配~这是一棵树, 树是二分图, 二分图最大权匹配多么容易~

然后去看题解, 换根DP? 吓得我赶紧关掉了......

哎哟, 简直是傻掉了......树上最大权匹配为什么要跑网络流......直接做DP不好嘛......

感觉换根DP这个名字是那位博主自己取的, 不过我觉得很贴切. 类似的方法和马老师还讨论过 (他也管这个过程叫换根, 题目是[树上所有点的最远点](/2017/04/12/hdu2196-computer/)), 我自己也总结过: [Codeforces Round #408 (Div. 2) C Bank Hacking](/2017/04/11/cf796/)

所以这道题的实际难度是CF Div.2 C题? 只是我做不出C题的时候居多啦......

具体而言, 本题这样搞:

以1为根, 跑一遍DP.

设dp[0/1][u]表示以u为根的子树的最大得分, u的父边不选/选. 令dp[2][u] = max { dp[0][u], dp[1][u] }, 则dp[0][u] = sum dp[2][v] (v is son of u), dp[1][u] = max { dp[0][u] + dp[0][v] - dp[2][v] + c(u, v) + c(u, fa[u]) (v is son of u) } (对式子进行了一点变形, 降低转移复杂度).

令mx[u] = max { dp[0][v] - dp[2][v] + c(u, v) }, se[u]为dp[0][v]-dp[2][v]+c(u,v)的次大. 则dp[1][v] = dp[0][u] + mx[u].

再跑一遍DP, 考虑以其他点为根的答案.

设当前DFS遍历到边(u,v), 为了得到以v为根的答案, 需要知道T-subtree(v)这部分的mx和dp[0].

先用T-subtree(fa[u])的信息更新mx[u], se[u]和dp[0][u]. 然后, 将mx[u]和se[u]中的一个作为mx[T-subtree(v)]传给v (具体选哪个取决于dp[0][v]-dp[2][v]+c(u,v)是否等于mx[u]); 将dp[0][u]-dp[0][v]作为dp[0][T-subtree(v)]传给v.

(用LaTEX排版可能会美观一些, 可是Mathjax加载得好慢TAT)

```cpp
const int N = 2e5, neg_inf = numeric_limits<int>::min();

struct Edge {
	int to, c;
};

vector<Edge> adj[N+1];
int ans, dp[3][N+1], s[N+1], mx[N+1], se[N+1];

void dfs(int u, int p)
{
	mx[u] = se[u] = neg_inf;
	rep (i, 0, adj[u].size()) {
		int v = adj[u][i].to, c = adj[u][i].c;
		if (v == p) continue;
		dfs(v, u);
		s[u] += dp[2][v] = max(dp[0][v], dp[1][v] += c);
		int t = dp[0][v] - dp[2][v] + c;
		if (t > mx[u]) se[u] = mx[u], mx[u] = t;
		else if (t > se[u]) se[u] = t;
	}
	dp[0][u] = s[u];
	dp[1][u] = mx[u] > neg_inf ? s[u] + mx[u] : neg_inf;
}

void dfs2(int u, int p, int f2, int t)
{
	s[u] += f2;
	if (t > mx[u]) se[u] = mx[u], mx[u] = t;
	else if (t > se[u]) se[u] = t;

	ans = max(ans, s[u]);
	
	rep (i, 0, adj[u].size()) {
		int v = adj[u][i].to, c = adj[u][i].c;
		if (v == p) continue;
		int f0 = s[u] - dp[2][v], x = dp[0][v] - dp[2][v] + c == mx[u] ? se[u] : mx[u],
			f2 = max(f0, x > neg_inf ? f0 + x + c: neg_inf);
		dfs2(v, u, f2, f0 - f2 + c);
	}
}

int main()
{
	int n;
	scanf("%d", &n);
	rep (i, 0, n-1) {
		int a, b, c;
		scanf("%d%d%d", &a, &b, &c);
		adj[a].push_back((Edge){b, c});
		adj[b].push_back((Edge){a, c});
	}
	dfs(1, 0);
	dfs2(1, 0, 0, neg_inf);
	printf("%d\n", ans);
	return 0;
}
```