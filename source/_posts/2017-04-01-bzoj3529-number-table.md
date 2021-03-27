---
title: "[bzoj 3529] [Sdoi2014]数表"
date: 2017-04-01 13:24:00
categories:
- bzoj
tags:
- 数论
- 莫比乌斯反演
- 树状数组
---
一种数表的第i行第j列等于能同时整除i和j的所有自然数之和. 求n\*m的数表所有不大于a的项之和对2^31取模的结果. Q组询问. (1&le;n,m&le;10^5, |a|&le;10^9, 1&le;Q&le;2\*10^4)
<!--more-->
如果没有a的限制, 则
$$
ans(n,m,\infty) = \sum_x x\lfloor \frac n x \rfloor \lfloor \frac m x\rfloor
$$

(事后诸葛亮: $x = \sum_{d\backslash x}\mu(\frac x d)\sigma(d)$ 但是这样没法把$a$搞进去)

然而这个式子对有a的限制的版本没什么帮助......大概得枚举$y=1,2,\ldots,a$, 统计有多少项满足$\sum_x x[x\backslash i][x\backslash j] = y$. 或许要用数据结构维护一下.

但是还是看了题解, 思路已经很接近了啊......

$\sum_x x[x\backslash i][x\backslash j] = \sigma(\gcd(i, j))$由$\gcd(i, j)$决定, 后者至多有10^5个取值. 而$\gcd(i,j)$对多少项有贡献是容易通过莫比乌斯反演计算的.

记$\sigma(x, a) = \sigma(x)[\sigma(x) \le a]$.

$$
\begin{array} {}
ans(n, m, a)
&= \sum_d \sigma(d, a) \sum_i \mu(i) \lfloor \frac n {id} \rfloor \lfloor \frac m {id} \rfloor \\
&= \sum_x \lfloor \frac n x\rfloor \lfloor \frac m x\rfloor \sum_{d\backslash x} \sigma(d, a)\mu(\frac x d)
\end{array}
$$

第二个变形的动机是, 分离一定无法预处理的项和可能可以预处理的项.

记$g(x, a) = \sum_{d\backslash x}\sigma(d, a) \mu(\frac x d)$, 则
$$
ans(n, m, a) = \sum_x \lfloor \frac n x\rfloor \lfloor \frac m x \rfloor g(x, a)
$$

筛法计算所有$\sigma$和$\mu$值, 将$(\sigma(x), x)$从小到大排序. 将询问离线化, 按照$a$从小到大排序, 用树状数组维护$g(x, a)$的前缀和.

时间复杂度$O(N\lg^2 N + Q\sqrt N\lg N)$.

感觉没搞出来的一个原因是没把$\sigma$抽象出来.

```cpp
const int N = 1e5, MAX_Q = 2e4;
typedef pair<int, int> ii;
typedef long long ll;
short mu[N + 1];
int sum[N + 1];

inline int lowbit(int x)
{
	return x & -x;
}

struct BIT {
	int v[N + 1];
	
	void add(int x, int a)
	{
		while (x <= N) {
			v[x] += a;
			x += lowbit(x);
		}
	}

	int query(int x)
	{
		int s = 0;
		while (x) {
			s += v[x];
			x -= lowbit(x);
		}
		return s;
	}
} T;

struct Query {
	int k, n, m, a;
	bool operator<(const Query& o) const
	{
		return a < o.a;
	}
} Q[MAX_Q];

void sieve()
{
	static bool notPrime[N + 1];
	static int prime[N + 1];
	static ll fac[N + 1];
	int cnt = 0;
	mu[1] = sum[1] = 1;
	For (i, 2, N) {
		if (!notPrime[i]) {
			prime[cnt++] = i;
			mu[i] = -1;
			sum[i] = i+1;
			fac[i] = 1-(ll)i*i;
		}
		for (int j = 0, p, x; j < cnt && (x = (p = prime[j])*i) <= N; ++j) {
			notPrime[x] = true;
			if (i % p) {
				mu[x] = -mu[i];
				sum[x] = sum[i] * (p+1);
				fac[x] = 1-(ll)p*p;
			} else {
				sum[x] = sum[i] * (fac[x] = fac[i]*p - p + 1) / fac[i];
				break;
			}
		}
	}
}

int solve(int n, int m)
{
	int ans = 0;
	for (int i = 1, j, ed = min(n, m); i <= ed; i = j+1) {
		int _n = n/i, _m = m/i;
		j = min(ed, min(n/_n, m/_m));
		ans += _n * _m * (T.query(j) - T.query(i-1));
	}
	return ans;
}

ii L[N];
int ans[MAX_Q];

int main()
{
	sieve();
	For (i, 1, N)
		L[i-1] = ii(sum[i], i);
	sort(L, L+N);
	int t;
	scanf("%d", &t);
	Rep (i, 0, t) {
		scanf("%d%d%d", &Q[i].n, &Q[i].m, &Q[i].a);
		Q[i].k = i;
	}
	sort(Q, Q+t);

	int j = 0;
	Rep (i, 0, t) {
		Query& q = Q[i];
		while (j < N && L[j].first <= q.a) {
			for (int x = L[j].second, k = 1; x <= N; x += L[j].second, ++k)
				T.add(x, L[j].first * mu[k]);
			++j;
		}
		ans[q.k] = q.a > 0 ? solve(q.n, q.m) : 0;
	}

	const int msk = (1LL << 31) - 1;
	Rep (i, 0, t) printf("%d\n", ans[i] & msk);
	return 0;
}
```