---
title: "[bzoj 2219] 数论之神"
date: 2017-03-26 17:17:05
categories:
- bzoj
tags:
- 数论
- 中国剩余定理
- BSGS
---
给定正整数A, B, K, 求$x^A \equiv B \pmod {2K+1}$在$x\in [0, 2K]$上的解的个数, T组数据. (T &le; 1000, 1 &le; A, B &le; 10^9, 1 &le; K &le; 5\* 10^8)
<!--more-->
我的做法好像不太主流, 后面会讲解两种方法.

# 背景知识
## 中国剩余定理
设$n=n_1n_2\cdots n_k$, $n_i$两两互质, 则$\mathbb Z_n$中的元素与$\mathbb Z_{n_1} \times \mathbb Z_{n_2} \times \cdots \mathbb Z_{n_k}$中的元素一一对应.

## 原根
*拉格朗日定理* 设$S'$是有限群$S$的子群, 则$|S'| \backslash |S|$.

有限群$S$中, 一个元素$a$不断搞自己会生成一个子群$\langle a\rangle$, 称$a$是$\langle a\rangle$的生成元, 满足$a^{(t)} = e$的最小正整数$t$是$a$在$S$中的阶, 记作$ord(a)$. 可以证明$ord(a) = |\langle a\rangle|$, 且$a^{(i)} = a^{(j)} \Leftrightarrow i \equiv j \pmod {ord(a)}$. 结合拉格朗日定理, 有$a^{(|S|)} = e$.

下面考虑模$n$乘法群$\mathbb Z_n^*$. 首先, 由上面子群有关性质, 可以导出欧拉定理.

若$ord(g)=|\mathbb Z_n^\*| = \varphi(n)$, 则$g$是整个群的生成元, 又称原根. 对所有奇素数$p$和所有正整数$e$, $\mathbb Z_n^* (n>1)$有原根当且仅当$n \in \left\\{2, 4, p^e, 2p^e \right\\}$.

$a$不是原根 <=> $ord(a)$是$\varphi(n)$的真约数 <=> 存在$\varphi(n)$的质因子$p$使得$ord(a) \backslash \frac {\varphi(n)} p$ <=> 存在$\varphi(n)$的质因子$p$使得$a^{\varphi(n)/p} \equiv 1$. 由此, 得到一个不那么暴力的检验原根的算法. 从小到大枚举$a\in \mathbb Z_n^*$, 检验之, 即可求得一个原根. 据说由于原根一般不会太大所以这样就可以了......

## 离散对数
$$a^x \equiv y \pmod n$$

当$\gcd(a, n) = 1$, 可以使用**大步小步**算法解这个关于$x$的方程.

设$m = \lfloor \sqrt \varphi(n) \rfloor$, $x = im - j\ (i=1,2,\ldots,\lceil \varphi(n)/m \rceil, j=1,2,\ldots,m)$, 方程转化为
$$(a^m)^i \equiv ya^j \pmod n$$

使用meet-in-middle的策略, 枚举$j$, 在哈希表中储存方程右边的所有取值, 再枚举$i$, 到哈希表中查询. 平均时间复杂度$O(\sqrt n)$.

## 同余式的一个性质
若$a \equiv b \pmod n$, 则$a,n$的公约数$d$也是$b$的约数. 再根据取模运算的分配律, 可以做除法: $a/d \equiv b/d \pmod n/d$.

## 模线性方程
$$ax \equiv b \pmod n$$

设$\gcd(a, n)=d$.
$0, a, 2a, \ldots, (n-1)a$是$0, d, 2d, \ldots, n-d$这$n/d$个数某个排列的$d$份复制. 于是上述方程有解当且仅当$d\backslash b$, 且如果有解, 则在$[0, n)$上有$d$个解.

# 两种做法公用的部分
首先将$2K+1$做质因数分解: $2K+1 = \prod_i p_i^{k_i}$, 则$x^A \equiv B \pmod {2K+1} \Leftrightarrow x^A \equiv B \pmod {p_i^{k_i}}$ (中国剩余定理). 考虑求这样的方程的解个数: $x^A \equiv B \pmod p^k$, $p$是奇素数 (所以$\mathbb Z_{p^k}^*$存在原根). 最后把答案乘起来即可.

先看一种特殊情形: $x^A \equiv B \pmod n$, $\mathbb Z_n^*$存在原根, $n, A$互质.

求这个方程在$[0, n)$上解的个数至少有两种方法.

- 取$\mathbb Z_n^*$的一个原根$g$, 设$x = g^t \bmod p^k, t=0,1,\ldots,\varphi(n)-1$ ($t$是唯一的), 则上述方程转化为${g^A}^t \equiv B \pmod n$, 对大步小步算法稍加改写, 以求得解的数目. 由于这里规定了$t$的范围, 所以模$\sqrt {\varphi(n)}$的最后一个周期直接枚举.

- 如果$B$和$n$不互质, 则无解. 还是取原根$g$, 设$x = g^{x'} \bmod p^k, B = g^b \bmod p^k$, 方程转化为$g^{Ax'} \equiv g^b \pmod n \Leftrightarrow Ax' \equiv b \pmod {\varphi(n)}$. 用大步小步算法求出$b$. $x'$在$[0, \varphi(n))$上的解与原方程的解一一对应. 根据*模线性方程*相关知识, 该方程有解当且仅当$gcd(A, n) \backslash b$, 如果有解, 解的数目是$gcd(A, n)$.

# 做法一
来自我自己.

若$x$与$p^k$互质, 用之前所述方法求之.

若$x$与$p^k$不互质, 则$p\backslash x$. 设$x = jp, j=0,1,\ldots,p^{k-1}-1$, 分类讨论.
- $A \ge k$. 方程左边等于0. 若$B\equiv 0$, 则有$p^{k-1}$个解, 否则无解.
- $A < k$. 根据上面所说的*同余式的一个性质*, $p^A \backslash B$, 如果不满足, 则无解; 否则, 方程的左右两边和模同时约去$p^A$: $j^A \equiv \frac B {p^A} \pmod p^{k-A}$, 这个问题和原问题有相同的形式, 可递归求解. 但是取值范围变了. 递归下去, $j=0,1,\ldots,p^{k-A}-1$. 根据周期性, 只需将答案乘上$p^{a-1}$.

问题顺利解决. 写代码的时候, 我把$B \equiv 0$在最前面单独讨论. $x^A \equiv 0 \pmod p^k \Leftrightarrow p^k \backslash x^A$, 于是$x$是范围内所有含因子$p^{\lceil k/A\rceil}$的数, 共$p^{k-\lceil k/A\rceil}$个.

# 做法二
参考
- [Po姐的题解](http://blog.csdn.net/popoqqq/article/details/41595187)
- [【BZOJ 2219】【超详细题解】数论之神 Regina8023](http://blog.csdn.net/regina8023/article/details/44863519).

思路大体相同, 中国剩余定理.

做法一讨论了$x$与$p^k$互质, $x$与$p^k$不互质两种情况. 事实上, $B$与$p^k$的关系已经决定了$x$与$p^k$是否互质.

若$B\equiv 0$, 直接返回$p^{k-\lceil k/A\rceil}$.

若$B\not\equiv 0$, 设$B = p^eb (\gcd(p, b)=1)$, 则$e < k$. 根据*同余式的一个性质*, 如果有解, 则$p^e \backslash x^A$, 并且由于右边不含$p$, 左边也不能含$p$, 故方程等价于
$$(\frac x {p^{\frac e A}}) \equiv b \pmod {p^{k-e}}$$
其中$\frac x {p^{\frac e A}}$与$p^{k-e}$互质.

和*做法一*中的递归类似, 这个方程解的个数要乘上$p^e$才是我们要的答案. 用*公用部分*所述方法求之.

# 感想
这道题的综合性比较强, 涉及了不少初等数论的知识. 细心分类讨论, 关注定义域的变化.

```cpp
#include <bits/stdc++.h>
#define Rep(i, a, b) for (int i = (a), i##_ = (b); i < i##_; ++i)
#define For(i, a, b) for (int i = (a), i##_ = (b); i <= i##_; ++i)
#define Down(i, a, b) for (int i = (a), i##_ = (b); i >= i##_; --i)

using namespace std;
typedef long long ll;
typedef pair<int, int> ii;
typedef vector<ii> Vec;
typedef Vec::iterator iter;

struct Hash {
	const static int M = 38321;
	vector<ii> h[M];
	int cnt, used[M];

	void insert(int v)
	{
		int k = v % M;
		if (h[k].empty()) used[cnt++] = k;
		for (iter x = h[k].begin(); x != h[k].end(); ++x)
			if (x->first == v) {
				++x->second;
				return;
			}
		h[k].push_back(ii(v, 1));
	}

	int operator[](int v)
	{
		int k = v % M;
		for (iter x = h[k].begin(); x != h[k].end(); ++x)
			if (x->first == v)
				return x->second;
		return 0;
	}
	
	void clear()
	{
		while (cnt) h[used[--cnt]].clear();
	}
} H;

void decompose(int n, Vec& v)
{
	for (int d = 2; d*d <= n; ++d) {
		if (n % d == 0) {
			ii t(1, d);
			do {
				t.first *= d;
				n /= d;
			} while (n % d == 0);
			v.push_back(t);
		}
	}
	if (n > 1)
		v.push_back(ii(n, n));
}

ll fp(ll x, ll n)
{
	ll y = 1;
	for (; n; n >>= 1, x *= x)
		if (n & 1)
			y *= x;
	return y;
}

ll fpm(ll x, ll n, ll m)
{
	ll y = 1;
	for (; n; n >>= 1, (x *= x) %= m)
		if (n & 1)
			(y *= x) %= m;
	return y;
}

int bsgs(ll a, ll y, ll n, ll r)
{
	H.clear();
	int m = round(sqrt(r)), cnt = 0;
	ll t = y;
	For (i, 1, m) // y*a^i, i=1,2,...,m
		H.insert((t *= a) %= n);
	ll d = fpm(a, m, n);
	t = 1;
	For (i, 1, r/m) // (a^m)^i, i=1,2,...,floor(r/m)
		cnt += H[(t *= d) %= n];
	Rep (i, r/m*m, r) // a^i, i=m*floor(r/m),...,r-1
		cnt += t == y, (t *= a) %= n;
	return cnt;
}

int primitive_root(int a, int p, int r)
{
	Vec v;
	decompose(r, v);
	Rep (g, 2, a) if (g % p) {
		bool ok = true;
		for (iter x = v.begin(); x != v.end(); ++x)
			if (fpm(g, r/x->second, a) == 1) {
				ok = false;
				break;
			}
		if (ok) return g;
	}
	assert(0);
	return -1;
}

int solve(int a, int b, int m, int p, int k)
{
	if (!b) return fp(p, k-(k+a-1)/a);
	int phi = m/p*(p-1), g = primitive_root(m, p, phi), ans = bsgs(fpm(g, a%phi, m), b, m, phi);
	if (a < k) {
		int t = fp(p, a), m1 = m / t;
		if (b % t == 0)
			ans += t / p * solve(a, b/t%m1, m1, p, k-a);
	}
	return ans;
}

int main()
{
	int T, K, A, B;
	scanf("%d", &T);
	while (T--) {
		scanf("%d%d%d", &A, &B, &K);
		Vec v;
		decompose(2*K + 1, v);
		int ans = 1;
		for (iter x = v.begin(); x != v.end(); ++x) {
			int m = x->first, p = x->second, k = 1;
			for (int t = p; t < m; t *= p) ++k;
			ans *= solve(A, B % m, m, p, k);
		}
		printf("%d\n", ans);
	}
	return 0;
}
```