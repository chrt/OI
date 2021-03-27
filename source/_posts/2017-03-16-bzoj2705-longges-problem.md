---
title: "[bzoj 2705] [SDOI2012] Longge的问题"
date: 2017-03-16 22:55:29
categories:
- bzoj
tags:
- 数论
- 欧拉函数
---
求∑gcd(i, N)(1&le;i&le;N). (0&lt;N&le;2^32)
约数个数有神奇的渐近上界.
<!--more-->
gcd(i,N)是N的因子, 而一个数的因子个数是$O(\sqrt n)$的. 枚举N的因子d, 看看有多少个i满足gcd(i,N)=d, 再把数目乘上d. d也是i的因子, 不妨枚举i/d, 这样, 条件转化为互质 - 多么亲切.
$$
\begin{split}
\sum_{i=1}^N gcd(i,N)
&= \sum_{d \backslash N} d\sum_{i=1}^N [gcd(i,N)=d] \\
&= \sum_{d \backslash N} d\sum_{i=1}^{N/d} [gcd(i, N/d) = 1] \\
&= \sum_{d \backslash N} d\varphi(N/d)
\end{split}
$$

这里求欧拉函数用公式$\varphi(n)=n\prod_i (1-1/p_i)$, 时间复杂度$O(\sqrt n)$.

忽然发现这个算法的时间复杂度很迷......显然有一个上界$O(n)$, 又显然没达到这个上界. 搞清楚了再来更新.

---
发现vfk的一篇blog: [<除数函数的渐近上界?>](http://vfleaking.blog.163.com/blog/static/174807634201341913040467/) 原来约数个数的幂和有一个学名: 除数函数.

约数个数$\tau(n)$的增长渐近意义下慢于所有多项式函数. 根据观察, 这个公式靠谱:
$$\tau(n) < n^{frac {1.066} {\ln\ln n}}, n \ge 3$$

当$n=3$时右边约为225862. 但后面渐渐就正常了.

就具体数值而言, 引用一下上文的实验结果:
10^9以内最大的Highly composite number是735134400，约数个数为1344个。(1000)
int32以内最大的Highly composite number是2095133040，约数个数为1600个。(1000)
10^18以内最大的Highly composite number是897612484786617600，约数个数为103680个。(10^5)
int64以内最大的Highly composite number是9200527969062830400，约数个数为161280个。(10^5)

感谢vfk!

也就是说, $2\sqrt n$是约数个数很松的一个上界. int范围内, 枚举约数, 再对每个约数干点根号级别的事情是可以接受的.

---

```cpp
#include <bits/stdc++.h>

using namespace std;
typedef long long ll;

ll phi(ll n)
{
	ll ret = n;
	for (ll i = 2; i*i <= n; ++i)
		if (n % i == 0) {
			ret = ret / i * (i-1);
			do {
				n /= i;
			} while (n % i == 0);
		}
	if (n > 1)
		ret = ret / n * (n-1);
	return ret;
}

int main()
{
	ll n, ans = 0;
	scanf("%lld", &n);
	for (ll i = 1; i*i <= n; ++i) if (n % i == 0) {
		ll j = n/i;
		ans += i * phi(j) + (i != j ? j * phi(i) : 0);
	}
	printf("%lld\n", ans);
	return 0;
}
```