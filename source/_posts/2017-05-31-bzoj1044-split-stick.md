---
title: "[bzoj 1044] [HAOI2008]木棍分割"
date: 2017-05-31 12:48:24
categories:
- bzoj
tags:
- 二分
- 动态规划
---
n根木棍长度为Li依次连接在一起, 最多砍断m个连接处, 问长度最大的一段的最小长度, 并求出使长度最大的一段最短的方案数模10007. (n&le;5\*10^4, 0&le;m&le;min(n-1,1000), 1&le;Li&le;1000)
<!--more-->
第一问就是 NOIP 2015 D2 T1, 二分, 贪心地验证.

第二问DP. 由于最大长度已知, 所以状态只有两维: $f(i,j)$表示前i根木棍, 切割j次, 每一部分长度不大于$l_0$的方案数.

以为长度的最大值是否取得到$l_0$是一个问题, 所以下面的代码$f(0,i,j)$表示最大长度严格小于$l_0$的方案数, $f(1,i,j)$表示最大长度等于$l_0$的方案数.

事实上, 并不存在这个问题, 因为$l_0$是最大长度的最小值.

$$
f(i,j) = \sum_{0<k<i} f(k,j-1) [L[k+1..i]\le l_0]
$$

满足$L[k+1..i]\le l_0$的$k$是连续的一段, 并且左端点随$i$非严格单调递增. 用two-pointer扫一扫, 并维护这个区间内$f$值之和 (在外层枚举$j$, 内层枚举$i$, 就不用开个数组把前缀和记录下来了).

遭遇0s TLE的惨案......数组得滚动一下.

```cpp
const int MOD = 10007, N = 5e4, M = 1e3;

// f[0]: max < r, f[1]: max = r
// g[0][i]: f[0][0..i], g[1][i] : f[1][0..i]

int n, m, l, r, a[N+1], s[N+1], f[2][2][N+1], g[2][2][N+1];

bool check(int c)
{
	for (int x = 0, y = 0, cnt = -1; x < n; x = y) { // (x, y]
		while (y <= n && s[y] - s[x] <= c) ++y;
		--y;
		if (++cnt > m) return false;
	}
	return true;
}

void bs()
{
	while (r-l > 1) { // (l, r]
		int m = (l+r)/2;
		if (check(m)) r = m;
		else l = m;
	}
}

int solve()
{
	rep (y, 1, n+1) {
		g[0][0][y] = g[0][0][y-1] + (f[0][0][y] = s[y] < r) % MOD;
		g[1][0][y] = g[1][0][y-1] + (f[1][0][y] = s[y] == r) % MOD;
	}

	int ans = f[1][0][n];
	
	rep (j, 1, m+1) {
		int t = j & 1;
		for (int y = 1, x = 0; y <= n; ++y) {
			while (s[y] - s[x] > r) ++x;

			f[1][t][y] = ((s[y]-s[x] == r ? f[0][t^1][x] : 0) + g[1][t^1][y-1] - (x ? g[1][t^1][x-1] : 0)) % MOD;
			f[0][t][y] = (g[0][t^1][y-1] - g[0][t^1][x] + (s[y]-s[x] < r ? f[0][t^1][x] : 0)) % MOD;

			g[1][t][y] = (g[1][t][y-1] + f[1][t][y]) % MOD;
			g[0][t][y] = (g[0][t][y-1] + f[0][t][y]) % MOD;
		}
		(ans += f[1][t][n]) %= MOD;
	}

	return (ans + MOD) % MOD;
}

int main()
{
	scanf("%d%d", &n, &m);
	rep (i, 1, n+1) {
		scanf("%d", a+i);
		s[i] = s[i-1] + a[i];
		l = max(l, a[i]);
	}
	--l;
	r = s[n];
	bs();
	printf("%d %d\n", r, solve());
	return 0;
}
```