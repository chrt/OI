---
title: "[bzoj 1030] 文本生成器"
date: 2017-04-14 20:54:43
categories:
- bzoj
tags:
- AC自动机
- 动态规划
---
字符集为全体大写英文字母. 给N个串, 问存在多少个长度为M的文本包含其中至少一个串. 答案模10007. (N&le;60, M和所有串的长度&le;100)
<!--more-->
和 [HNOI2008]GT考试 类似. 单串AC自动机变成真的AC自动机, 但减少了矩阵快速幂的难度.

对N个串建立AC自动机, 把所有边补齐, 则答案等于从根出发, 长度为M且不包含给定串的路径数目. *不包含给定串* 该做何理解呢? 用AC自动机读入一个串, 停在节点x, 则我们匹配到了x及x在fail树上所有祖先中的单词节点. 因此, 首先, 路径不能经过单词节点. 然后, 如果某一节点通过后缀链接能走到单词节点, 则该节点也禁用. 递推求出这些被禁止的点, 做DP即可. `dp[i][j]`表示长度为i, 终止于j, 且不经过禁用节点的路径数目.

```cpp
const int SIGMA = 26, N = 60*100 + 2, M = 100, MOD = 10007;

struct AC {
	int ch[N][SIGMA], f[N], size;
	bool word[N];

	AC(): size(1) {}
	
	void insert(char s[])
	{
		int now = 1;
		for (int i = 0; s[i]; ++i) {
			int& c = ch[now][s[i]-'A'];
			if (!c) c = ++size;
			now = c;
		}
		word[now] = true;
	}

	void build()
	{
		queue<int> Q;
		f[1] = 1;
		rep (i, 0, SIGMA) {
			int& v = ch[1][i];
			if (v)
				f[v] = 1, Q.push(v);
			else
				v = 1;
		}
		
		while (!Q.empty()) {
			int u = Q.front();
			Q.pop();
			rep (i, 0, SIGMA) {
				int& v = ch[u][i];
				if (v)
					word[v] |= word[f[v] = ch[f[u]][i]], Q.push(v);
				else
					v = ch[f[u]][i];
			}
		}
	}
} ac;

int dp[M+1][N];

int solve(int m)
{
	dp[0][1] = 1;
	rep (i, 0, m) {
		rep (j, 1, ac.size + 1) {
			if (ac.word[j]) continue;
			rep (k, 0, SIGMA)
				(dp[i+1][ac.ch[j][k]] += dp[i][j]) %= MOD;
		}
	}
	int ans = 0;
	rep (i, 1, ac.size + 1)
		if (!ac.word[i])
			(ans += dp[m][i]) %= MOD;
	return ans;
}

int main()
{
	int n, m;
	scanf("%d%d", &n, &m);
	while (n--) {
		char s[101];
		scanf("%s", s);
		ac.insert(s);
	}
	ac.build();
	int ans = 1;
	rep (i, 0, m) (ans *= SIGMA) %= MOD;
	(ans -= solve(m) - MOD) %= MOD;
	printf("%d\n", ans);
	return 0;
}
```