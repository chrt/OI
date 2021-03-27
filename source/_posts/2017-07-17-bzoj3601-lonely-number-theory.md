---
title: "[bzoj 3601] 一个人的数论"
date: 2017-07-17 17:20:46
categories:
- bzoj
tags:
- 莫比乌斯反演
- 高斯消元
---
求小于 $n$ 且与 $n$ 互质的正整数的 $k$ 次方之和 $f_k(n)$. $n$ 以标准质因数分解的形式给出, 即 $n = \prod_{i=1}^w p_i^{\alpha_i}$. $(1\le w\le 1000, 2\le p_i,\alpha_i\le 10^9)$
<!--more-->
记 $S_k(n) = \sum_{i=1}^n i^k$, 把 $\sum_{d\backslash n} \mu(d) = [n = 1]$ 代入 $[gcd(i,n) = 1]$, 得
$$
f_k(n) = \sum_{d\backslash n} \mu(d) d^k S_k(\frac n d)
$$
接下来, 推测要从积性函数的角度入手......可是根据观察, $f_k$ 并不是积性的.

只差一步......

关于 $S_k$, 我们知道什么? 它是一个 $k+1$ 次多项式, 系数可以由高斯消元得到. 设 $S_k(n) = \sum_{i=0}^{k+1} a_i n^i$, 那么
$$
f_k(n) = \sum_{i=0}^{k+1} a_i\sum_{d\backslash n}\mu(d) d^k (\frac n d)^i
$$

记 $h_i(n) = \sum_{d\backslash n}\mu(d) d^k (\frac n d)^i$. 它是 $\mu(n)n^k, n^i$ 两个积性函数的狄利克雷卷积, 所以它也是积性的.

$$
h_i(p^\alpha) = \sum_{j=0}^\alpha \mu(p^j) {(p^j)}^k {(p^{\alpha-j})}^i
$$
莫比乌斯函数当且仅当自变量是`square-free-number`时非 0, 所以
$$
h_i(p^\alpha) = p^{\alpha i}(1 - p^{k-i})\\\\
h_i(n) = n^i \prod_{j=1}^w (1-p_j^{k-j})
$$

于是, 忽略快速幂的时间复杂度, 本题以 $O(k^3 + kw)$ 的时间解决.

注意, $S_k(n)$ 是 $k+1$ 次多项式, 需要 $k+2$ 个方程确定. (当然, 已知 $a_0 = 0$)

```cpp
typedef long long ll;

const int W = 1000, K = 100, MOD = 1e9 + 7;

ll fpm(ll x, int n)
{
	ll y = 1;
	for (; n; n >>= 1, (x *= x) %= MOD)
		if (n & 1)
			(y *= x) %= MOD;
	return y;
}

struct Gauss
{
	ll a[K+1][K+2];
	void main(int n)
	{
		rep (i, 0, n)
		{
			int r = i;
			while (r < n && !a[r][i]) ++r;
			assert(r != n);
			if (r != i) rep (k, i, n+1) swap(a[i][k], a[r][k]);
			ll t = fpm(a[i][i], MOD-2);
			rep (k, i, n+1) (a[i][k] *= t) %= MOD;
			rep (j, i+1, n) if (a[j][i])
				per (k, n, i) (a[j][k] -= a[j][i] * a[i][k]) %= MOD;
		}
		per (i, n-2, 0) rep (j, i+1, n+1)
			(a[i][n] -= a[i][j] * a[j][n]) %= MOD;
	}
} G;

int h[K+1];

int main()
{
	int k, w, ans = 0, n = 1, p, a;
	scanf("%d%d", &k, &w);
	fill_n(h, k+1, 1);
	rep (i, 0, w)
	{
		scanf("%d%d", &p, &a);
		n = n * fpm(p, a) % MOD;
		ll t = fpm(p, MOD-2) % MOD;
		per (j, k, 0)
			h[j] = 1LL * h[j] * (1-t) % MOD, (t *= p) %= MOD;
	}
	ll m = n;

	rep (i, 0, k+1)
		h[i] = h[i] * m % MOD, (m *= n) %= MOD;

	rep (i, 0, k+1)
	{
		G.a[i][0] = i+1;
		rep (j, 1, k+1) G.a[i][j] = G.a[i][j-1] * (i+1) % MOD;
		G.a[i][k+1] = i ? (G.a[i-1][k+1] + G.a[i][k-1]) % MOD : 1;
	}
	G.main(k+1);
	rep (i, 0, k+1) ans = (ans + G.a[i][k+1] * h[i]) % MOD;
	printf("%d\n", (ans + MOD) % MOD);
	return 0;
}
```
