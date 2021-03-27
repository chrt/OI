---
title: "[bzoj 1042] [HAOI2008]硬币购物"
date: 2017-05-31 19:40:10
categories:
- bzoj
tags:
- 动态规划
- 容斥原理
---
4种面值分别为ci的硬币, tot次购物: 第i种硬币带di枚, 买价值为s的物品, 有多少种付款方法? (di,s&le;10^5,tot&le;1000)
<!--more-->
大概好几个月以前, 听说这道题是容斥, 尝试失败 (并没有看到硬币购物和三个圈圈之间的联系 TAT), 然后去看题解.

嗯, 今天把题解实现了一下......

设硬币的种类为$k$.

有数量限制, 强行DP的时间复杂度是 $O(ks^2)$. 但是如果没有数量限制, 可以 $O(ks)$. 那么, 先求出没有限制的答案, 再减去不合法的, 再加上减多了的, 再......

设硬币没有数量限制, 买价值为 $i$ 的物品有 $f(i)$ 种付款方式. 当 $i<0$, $f(i)=0$. 怎样计算第1种硬币超过 $d_1$ 枚的方案数呢? 它就等于 $f(s-(d_1+1)c_1)$.

$t$ 种硬币的数目超出限制, 容斥系数是 $(-1)^t$.

时间复杂度 $O(ks + k 2^k tot)$.

```cpp
typedef long long ll;

const int S = 1e5;

ll s, c[4], d[4], f[S+1];

int main()
{
	int tot;
	f[0] = 1;
	rep (i, 0, 4) {
		scanf("%lld", c+i);
		rep (j, c[i], S+1)
			f[j] += f[j-c[i]];
	}
	scanf("%d", &tot);
	while (tot--) {
		rep (i, 0, 4)
			scanf("%lld", d+i);
		scanf("%lld", &s);
		ll ans = 0;
		rep (i, 0, 1<<4) {
			ll t = s;
			int sgn = 1;
			rep (j, 0, 4)
				if (i & (1<<j))
					t -= (d[j]+1) * c[j], sgn *= -1;
			if (t >= 0) ans += sgn * f[t];
		}
		printf("%lld\n", ans);
	}
	return 0;
}
```