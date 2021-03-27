---
title: "[bzoj 2186] [Sdoi2008] 沙拉公主的困惑"
date: 2017-03-21 17:02:39
categories:
- bzoj
tags:
- 数论
---
求[1,n!]内与m!互质的整数的数目, T组数据, 答案对同一个素数R取模. (1&le;m&le;n&le;10^7, T&le;10^4, R&le;10^9+10)
<!--more-->
由于n!非常大, [1,n!]内数的信息难以直接获取, 所以考虑用总数-与m!不互质的数.

不互质, 就是有大于1的公因子, 或者说素因子. m!有哪些素因子是容易获知的 - 就是[1,m]内的所有素数. m!的因子均可以整除n! (大概这就是题目以阶乘作为条件的原因). 运用容斥原理 (虽然直接容斥复杂度是指数级别的, 但仍然尝试推导出公式), 有:
$$ans = n!(1 - \frac 1 2 - \frac 1 3 - \frac 1 5 - \cdots + \frac 1 6 + \frac 1 10 + \cdots)$$

咋和欧拉函数的公式这么像呢? 将括号里那一串收缩成$(1-1/p_1)(1-1/p_2)\cdots(1-1/p_k), p_i\text{是}m!\text{的素因子}$.
$$ans = n!\frac {(p_1-1)(p_2-1)\cdots(p_k-1)} {p_1p_2\cdots p_k}$$

答案对R取模, R可能是$p_i$中的一个, 所以不能直接求逆元. 如果R是$p_i$中的一个, 那么它必定也是$n!$的因子. 预处理$n!, \prod_i (p_i-1), \prod_i p_i$, 但是$n!, \prod_i p_i$不把R乘进去, 分类讨论:
- $m < R, n < R$ 没有影响.
- $m < R, n \ge R$ 答案为0.
- $m \ge R$ 没有影响.

问题解决.

看了下别人的题解, 很多都忽略了R是$p_i$因子的可能性. 出题人造数据的时候没有考虑到这个错误......还是说, 他觉得没必要去卡?

还发现公式的另一种推导. [1,m!]内与m!互质的数有$\varphi(m!)$个. 由于$gcd(x,y)=gcd(x\mod y,y)$ (辗转相除的原理), 并且n!是m!的倍数, 所以[1,n!]每m!分一段, 每一段均有$\varphi(m!)$个数与m!互质:
$$ans = \frac {n!} {m!} \varphi(m!) = n!\prod_i (1-\frac 1 {p_i}), p_i\text{是}m!\text{的素因子}$$

写了一下埃氏筛法. 没有预处理逆元.

总的时间复杂度$O(n\lg\lg n + T\lg R)$.

```cpp
#include <bits/stdc++.h>
#define For(i, a, b) for (int i = (a), i##_ = (b); i <= i##_; ++i)

using namespace std;
typedef long long ll;
const int N = 1e7;
int R, f[N + 1], g[N + 1], h[N + 1];
bool b[N + 1];

void init()
{
	f[0] = 1;
	For (i, 1, N) f[i] = (ll)f[i-1] * (i == R ? 1 : i) % R;
	for (int i = 2; i*i <= N; ++i)
		if (!b[i])
			for (int j = i, t; (t = i*j) <= N; ++j)
				b[t] = true;
	h[1] = g[1] = 1;
	For (i, 2, N) {
		g[i] = (ll)g[i-1] * (b[i] ? 1 : i-1) % R;
		h[i] = (ll)h[i-1] * (b[i] || i == R ? 1 : i) % R;
	}
}

ll inv(ll x)
{
	ll y = 1;
	for (int n = R-2; n; (x *= x) %= R, n >>= 1)
		if (n & 1)
			(y *= x) %= R;
	return y;
}

int main()
{
	int T;
	scanf("%d%d", &T, &R);
	init();
	while (T--) {
		int n, m;
		scanf("%d%d", &n, &m);
		printf("%lld\n", m < R && n >= R ? 0 : (ll)f[n] * g[m] % R * inv(h[m]) % R);
	}
	return 0;
}
```