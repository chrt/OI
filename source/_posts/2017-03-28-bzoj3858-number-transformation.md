---
title: "[bzoj 3858] Number Transformation"
date: 2017-03-28 11:53:17
categories:
- bzoj
tags:
- 数学
---
初始一个正整数x, 第i次操作将x变为i的不小于x的最小倍数, 问k次操作后得到多少. 多组数据. (1&le;x,k&le;10^10)
<!--more-->
第$i$次操作得到的数:
$$
x_i = \lceil \frac {x_{i-1}} i \rceil i
$$

或者展开, 以$k=3$为例:
$$
x_3 = \lceil \frac {\lceil \frac {\lceil \frac {x_0} 1\rceil \times 1} 2 \rceil \times 2} 3\rceil \times 3
$$

打表发现$\left\\{x\right\\}$会渐渐变成等差数列. 定义$y_i = x_i / i$, 则
$$
y_i = \lceil \frac {i-1} i y_{i-1}\rceil
$$

发现$\left\\{y\right\\}$单减. 由于当$i\rightarrow \infty$时, $\frac {i-1} i \rightarrow 1$, 猜测$\left\\{y\right\\}$*收敛*于某一个值. 打表发现的确如此, 而且只需要操作约$O(\sqrt x)$次. 按照这个思路写个程序提交就可以AC了.

但这是为什么呢? [BZOJ3858 Number Transformation - Xs酱~ - 博客园](http://www.cnblogs.com/rausen/p/4297717.html)

$$
y_i = y_{i+1} \\
\Leftrightarrow y_i = \lceil \frac i {i+1} y_i \rceil \\
\Leftrightarrow y_i-1 < \frac i {i+1} y_i \le y_i \\
\Leftrightarrow 0 \le y_i \le i \\
\Leftrightarrow i \ge \sqrt {x_i}
$$

其实这个推理并不能说明问题, 因为$x_i$在变大......能力有限.

```cpp
#include <bits/stdc++.h>

using namespace std;
typedef long long ll;

int main()
{
	ll x, k;
	int T = 0;
	while (scanf("%lld%lld", &x, &k) == 2 && x) {
		ll i, y = 0;
		for (i = 1; i <= k; ++i) {
			x = (x+i-1)/i;
			if (x == y) break;
			else y = x;
			x *= i;
		}
		printf("Case #%d: %lld\n", ++T, x == y ? x*k : x);
	}
	return 0;
}
```