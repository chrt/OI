---
title: "[bzoj 2154] Crash的数字表格"
date: 2017-04-01 14:50:27
categories:
- bzoj
tags:
- 数论
- 莫比乌斯反演
---
求$\sum_{i=1}^n\sum_{j=1}^m lcm(i, j) \bmod 20101009$. (n,m &le; 10^7)
<!--more-->
$$
\begin{array} {}
ans(n,m)
&= \sum_{i=1}^n\sum_{j=1}^m \frac {ij} {\gcd(i,j)} \\
&= \sum_d \frac 1 d \sum_{i=1}^n\sum_{j=1}^m ij[\gcd(i, j) = d] \\
&= \sum_d d \sum_{i=1}^{\lfloor n/d \rfloor}\sum_{j=1}^{\lfloor m/d \rfloor} ij[\gcd(i,j)=1]
\end{array}
$$
$\gcd$等于某个定值的限制不好搞, 但$\gcd$是某定值的倍数比较容易处理.
令
$$
f(k,n,m) = \sum_{i=1}^n\sum_{j=1}^m ij[k=\gcd(i,j)] \\\\
F(k,n,m) = \sum_{i=1}^n\sum_{j=1}^m ij[k\backslash\gcd(i,j)]
$$
则
$$
F(k,n,m) = \sum_{k\backslash x} f(x,n,m) = (\sum_{k\backslash i} i[i\le n])(\sum_{k\backslash j} j[j\le m])
= \frac {k^2} 4 \lfloor \frac n k \rfloor(1+\lfloor \frac n k \rfloor) \lfloor \frac m k \rfloor(1+\lfloor \frac m k \rfloor)
$$
反演一下, 令$k=1$, 并代回原式
$$
ans(n,m) = \frac 1 4\sum_d d \sum_x \mu(x)x^2 \lfloor \frac {n/d} x \rfloor(1+\lfloor \frac {n/d} x \rfloor) \lfloor \frac {m/d} x \rfloor(1+\lfloor \frac {m/d} x \rfloor)
$$
枚举商分段计算即可.

$O(n\lg n)$没有特殊的常数优化是跑不了一千万的......NOIP蚯蚓那题没能给我深刻的教训 QAQ

用积分估计一下时间复杂度, $O(n)$. 计算和式的部分的复杂度可能低于$\Theta(n)$, 但是不会算......

```cpp
typedef long long ll;
const int MOD = 20101009, QUARTER = 15075757, N = 1e7;
short mu[N + 1];
int sum[N + 1];

void sieve(int n)
{
	static bool notPrime[N + 1];
	static int prime[N + 1];
	int cnt = 0;
	sum[1] = mu[1] = 1;
	For (i, 2, n) {
		if (!notPrime[i]) {
			prime[cnt++] = i;
			mu[i] = -1;
		}
		for (int j = 0, x, p; j < cnt && (x = i*(p = prime[j])) <= n; ++j) {
			notPrime[x] = true;
			if (i % p)
				mu[x] = -mu[i];
			else
				break;
		}
		sum[i] = (sum[i-1] + (ll)i*i*mu[i] % MOD + MOD) % MOD;
	}
}

int f(int n, int m)
{
	ll ans = 0;
	for (int i = 1, j; i <= n; i = j+1) {
		int _n = n/i, _m = m/i;
		j = min(n, min(n/_n, m/_m));
		(ans += (ll)(sum[j] - sum[i-1] + MOD) * _n % MOD * (1+_n) % MOD * _m % MOD * (1+_m) % MOD) %= MOD;
	}
	return (ans * QUARTER) % MOD;
}

int main()
{
	int n, m;
	scanf("%d%d", &n, &m);
	if (n > m) swap(n, m);
	sieve(n);
	ll ans = 0;
	for (int i = 1, j; i <= n; i = j+1) {
		int _n = n/i, _m = m/i;
		j = min(n, min(n/_n, m/_m));
		(ans += (ll)(i+j)*(j-i+1)/2 % MOD * f(_n, _m)) %= MOD;
	}
	printf("%lld\n", ans);
	return 0;
}
```