---
title: "[bzoj3864] Hero meet devil：DP套DP"
date: 2017-01-29 12:16:55
categories:
- bzoj
tags:
- 动态规划
- 状压DP
- 计数
---
题意：对0&le;i&le;n，求有多少个长为m的DNA序列与长为n的DNA序列s的最长公共子序列长度为i。（n&le;15，m&le;1000）

换言之，设`t[1..i]`与`s[1..j]`的LCS为`d[i][j]`，求使`d[m][n]=x`的方案数。
<!-- more -->
关于DP套DP，先从陈老师 WC 2015《计数问题选讲》中摘一段：
```
简单地说就是通过一个外层的dp来计算使得另一个dp方程（子dp）最终结果为特定值的输入数。
（我坦白这名字是我瞎YY的）
那么，这个问题要怎么解决呢，我们不妨一位一位确定子dp的输入，不妨考虑已经枚举了前i位，
那么注意到，由于我们只对dp方程的最终结果感兴趣，我们并不需要记录这前i位都是什么，
只需记录对这前i位进行转移以后，dp方程关于每个状态的值就可以了（这个的意思是，外层dp的状态是所有子dp的状态的值）。
```

本题是陈老师出的，然后他以这道题为第一个例子。

先复习一下LCS的求法。（字符串问题下标从1开始比较方便，这样可以用0表示空串，简化边界；可惜我写完本题才认识到）

```
let n = length[s], m = length[t], d[0..m][0..n] be 2-D array of integers

LCS(s, t)
    d[0][0] = 1
    d[0][1..n] = 0
    for i=1 to m
        transform(s, i)
    return d[m][n]

transform(s, i)
    let c = t[i]
    for j=1 to n
        if s[j] == c
            d[i][j] = d[i-1][j-1] + 1
        else
            d[i][j] = max(d[i-1][j], d[i][j-1])
```

不妨以行为单位递推。如果知道`d[i][*]`、`s`和`c`，就能用`transform`过程求出`d[i+1][*]`。也许DP转移方程不一样，比如还可以写成`d[i][j] = max(d[i-1][j], d[i][j-1], d[i-1][j-1] + [t[i] = s[j]])`，方案不同，但`d`数组的取值是唯一的。

设`f[i][S]`为使`d[i][*]`为状态`S`的方案数，枚举`t[i]`，用加法原理以贡献的方式向`f[i+1][trans[S][t[i]]]`转移即可。

考虑怎么表示`S`。`d[i][j]`与`d[i][j-1]`相比，不会减少，且至多增加1。

证明：不等式`d[i][j]>=d[i][j-1]`总是成立。如果`d[i][j]`的一个最优解中`s[j]`没有匹配，则`d[i][j-1]>=d[i][j]`；如果所有`s[j]`都匹配，则`d[i][j-1]>=d[i][j]-1`。于是，`0<=d[i][j]-d[i][j-1]<=1`。

所以把`d[i][*]`差分一下，用一个0-1串记录即可。也就是说，状态`S`至多只有`2^n`种。事实上，随便输了几个串试验，只到达了几千个状态......挺稀疏的。

怎么求`trans[S][c]`？把`S`解码，使用`transform`过程即可。

设$\Sigma$为字符集。如果预处理转移，时间复杂度$O((n+m)|\Sigma|2^n)$。需要使用滚动数组优化空间，空间复杂度$O(|\Sigma|2^n)$。

[BZOJ 3864 Hero meet devil DP套DP - 世界 - 博客频道 - CSDN.NET](http://blog.csdn.net/popoqqq/article/details/46826271)

```cpp
#include <cstdio>
#include <cstring>
#include <algorithm>
using namespace std;
const int MAX_N = 15, MOD = 1e9+7;
const char* sigma = "ACGT";
int n, m, trans[1<<MAX_N][4], f[2][1<<MAX_N], ans[MAX_N+1];
char s[MAX_N+1];

void pretreatment()
{
	static int d[MAX_N], t[MAX_N];
	for (int S = 0; S < (1<<n); ++S) {
		d[0] = S & 1;
		for (int i = 1; i < n; ++i)
			d[i] = d[i-1] + ((S & (1<<i)) >> i);
		for (int i = 0; i < 4; ++i) {
			for (int j = 0; j < n; ++j) {
				if (s[j] == sigma[i])
					t[j] = 1 + (j ? d[j-1] : 0);
				else
					t[j] = max(j ? t[j-1] : 0, d[j]);
			}
			trans[S][i] = t[0];
			for (int j = 1; j < n; ++j)
				trans[S][i] = trans[S][i] | ((t[j]-t[j-1])<<j);
		}
	}
}

inline int count(int x)
{
	int ret = 0;
	while (x)
		++ret, x -= x&-x;
	return ret;
}

void dp()
{
	static int t[MAX_N];
	int now = 0;
	fill_n(f[0], 1<<n, 0);
	for (int j = 0; j < 4; ++j) {
		for (int i = 0; i < n; ++i)
			t[i] = s[i] == sigma[j] ? 1 : (i ? t[i-1] : 0);
		int S = t[0];
		for (int i = 1; i < n; ++i)
			S = S | ((t[i]-t[i-1])<<i);
		++f[0][S];
	}
	for (int i = 1; i < m; ++i) {
		fill_n(f[now^1], 1<<n, 0);
		for (int S = 0; S < (1<<n); ++S)
			for (int j = 0; j < 4; ++j)
				(f[now^1][trans[S][j]] += f[now][S]) %= MOD;
		now ^= 1;
	}
	fill_n(ans, n, 0);
	for (int S = 0; S < (1<<n); ++S)
		(ans[count(S)] += f[now][S]) %= MOD;
}

int main()
{
	int T;
	scanf("%d", &T);
	while (T--) {
		scanf("%s %d", s, &m);
		n = strlen(s);
		pretreatment();
		dp();
		for (int i = 0; i <= n; ++i)
			printf("%d\n", ans[i]);
	}
	return 0;
}
```