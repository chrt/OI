---
title: "[bzoj 3992] [SDOI2015]序列统计"
date: 2017-04-20 16:07:22
categories:
- bzoj
tags:
- FFT
- 生成函数
---
从一个元素均为小于m的非负整数的集合S中有顺序地取n个数 (可重复选取), 问有多少种方案使得这些数的乘积模m等于x, 答案模1004535809. (1&le;n&le;10^9, 3&le;m&le;8000, m是质数, 1&le;x;&le;m-1, 输入保证S中元素不重复)
<!--more-->
发生了一件不太好的事情, 不小心瞥到题解中有 "原根" 二字......

由于m是奇质数, 原根存在. 对每个非零数取指标, 乘法转化为加法, 问题变成类似于背包的形式.

设原根为g. 令 ai=[S中存在g^i] (i=0,1,...,φ(m)-1), A(z) = Σ aiz^i, 则答案等于 Σ [z^y]A^n [y mod φ(m) = x] mod 1004535809. 多项式乘法可以用快速幂, 但是次数可能变得非常大, 而我们只需要次数模 φ(m) 等于 x 这些项前的系数之和 (循环卷积). 只须每次乘法用循环卷积替代普通卷积即可.

由于不会那个对任意长度的序列DFT的算法, 每次老老实实做乘法, 在系数表示下按定义累加.

由于本题答案对质数 1004535809 = 479\*2^21 + 1 取模, 用NTT替代FFT: 把所有主n次单位根换成g^(p-1)/n (g是模数的原根, 这里g=3), 所有运算改在模意义下进行.

```cpp
typedef long long ll;

const int M = 1004535809, N = 8000, E = 14, L = (1<<E) + 1;

ll fpm(ll x, ll n, ll m=M)
{
	ll y = 1;
	for (; n; n >>= 1, (x *= x) %= m)
		if (n & 1)
			(y *= x) %= m;
	return y;
}

inline ll mod(ll x)
{
	return x -= x >= M ? M : 0;
}

namespace Convol {
	const ll g[2] = {3, 334845270};
	ll w[2][E + 1];
	int n, inv_n, rev[L];
	
	void init(int _n, int s)
	{
		n = _n;
		inv_n = fpm(n, M-2);
		
		rep (i, 0, 2) {
			w[i][s] = fpm(g[i], (M-1)/n);
			per (j, s, 1) w[i][j-1] = w[i][j] * w[i][j] % M;
		}
		
		--s;
		rep (i, 1, n) rev[i] = (rev[i>>1]>>1) | ((i&1)<<s);
	}

	void ntt(ll a[], int d=0)
	{
		rep (i, 0, n)
			if (rev[i] < i)
				swap(a[rev[i]], a[i]);
		for (int i = 1, m = 1; m < n; ++i, m <<= 1)
			for (int j = 0; j < n; j += m<<1) {
				ll _w = 1;
				rep (k, 0, m) {
					ll t = _w * a[j+m+k] % M;
					a[j+m+k] = mod(a[j+k] - t + M);
					a[j+k] = mod(a[j+k] + t);
					(_w *= w[d][i]) %= M;
				}
			}
		if (d)
			rep (i, 0, n) (a[i] *= inv_n) %= M;
	}
}

using Convol::ntt;

void power(ll a[], ll b[], int n, ll k)
{
	int m = 1, s = 0;
	while (m < 2*n-1) m <<= 1, ++s;
	Convol::init(m, s);

	fill_n(b+1, m-1, 0);
	b[0] = 1;
	
	while (k) {
		ntt(a);
		
		if (k & 1) {
			ntt(b);
			rep (i, 0, m)
				(b[i] *= a[i]) %= M;
			ntt(b, 1);
			per (i, m-1, n) {
				b[i-n] = mod(b[i-n] + b[i]);
				b[i] = 0;
			}
		}
		
		rep (i, 0, m)
			(a[i] *= a[i]) %= M;
		ntt(a, 1);
		per (i, m-1, n) {
			a[i-n] = mod(a[i-n] + a[i]);
			a[i] = 0;
		}
		
		k >>= 1;
	}
}

void decompose(int x, vector<int>& L)
{
	for (int d = 2; d*d <= x; ++d)
		if (x % d == 0) {
			L.push_back(d);
			do {
				x /= d;
			} while (x % d == 0);
		}
	if (x > 1)
		L.push_back(x);
}

int get_primitive_root(int p)
{
	int phi = p-1;
	
	vector<int> L;
	decompose(phi, L);

	rep (i, 2, p) {
		bool ok = true;
		rep (j, 0, L.size())
			if (fpm(i, phi/L[j], p) == 1) {
				ok = false;
				break;
			}
		if (ok) return i;
	}

	assert(0);
}

int ind[N];
ll a[L], b[L];

int main()
{
	int n, m, x, s;
	scanf("%d%d%d%d", &n, &m, &x, &s);
	int g = get_primitive_root(m), t = 1;
	rep (i, 0, m-1) {
		ind[t] = i;
		(t *= g) %= m;
	}
	rep (i, 0, s) {
		int y;
		scanf("%d", &y);
		if (y) ++a[ind[y]];
	}
	power(a, b, m-1, n);
	printf("%lld\n", b[ind[x]]);
	return 0;
}
```