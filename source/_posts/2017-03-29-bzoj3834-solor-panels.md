---
title: "[POI 2014] Solor Panels"
date: 2017-03-29 18:14:16
categories:
- POI
tags:
- 数论
---
求Smin&le;x&le;Smax, Wmin&le;y&le;Wmax时gcd(x,y)的最大值. n组数据. (1&le;n&le;1000, 1&le;Smin&le;Smax&le;10^9, 1&le;Wmin&le;Wmax&le;10^9)
<!--more-->
gcd的最大值即这些数所有因子的最大值. 枚举因子d, d是$[a,b]$内某数的因子当且仅当
$$
(L = \lfloor \frac {a-1} d \rfloor) < (R = \lfloor \frac b d \rfloor)
$$
同时$(L, R]$内所有整数也是因子.

$O(\sqrt U)$地枚举因子, 分别把这些线段取出来, 排序, 取并, 再做个two-pointer把两者求交.

担心时间会爆掉. 而且不开O2的运行时间是开O2的6倍......还是AC了, 经过对比, 跑得不算慢. 可是大家的代码怎么这么短......

$\lfloor \frac n k\rfloor$只有$O(\sqrt n)$种取值, 见[[51nod 1225] 余数之和](/2017/03/29/51nod1225-modsum/).

如果某段数$x \in [l, r]$的$\lfloor \frac {Smax} x\rfloor, \lfloor \frac {Wmax} x\rfloor$分别相等, 把它们放到一起考虑. 因为只需要比一下大小就知道这个区间是否合法, 而且求的是最大值, 所以这一段取右端点$r$即可.

代码变短了, 可是跑的变慢了. rank 1的tangjz的代码又短又快, 就去[唐老师的github代码库](https://github.com/tangjz/acm-icpc/blob/master/lydsy/3834.cpp)学习了一下. 原来倒着枚举就好.

跑得还是没有唐老师快......

```cpp
int main()
{
    int T;
    scanf("%d", &T);
    while (T--) {
        int s1, s2, w1, w2;
        scanf("%d%d%d%d", &s1, &s2, &w1, &w2);
        --s1, --w1;
        int i = min(s2, w2);
        while (true) {
            if (s1/i < s2/i && w1/i < w2/i) {
                printf("%d\n", i);
                break;
            }
            i = max(s2/(s2/i + 1), w2/(w2/i + 1));
        }
    }
    return 0;
}
﻿```

原来的代码:
```cpp
typedef pair<int, int> Seg;
const int N = 63250, inf = 1e9 + 1;
Seg s[N], w[N], t[N];
int ns, nw;

void cal(int st, int ed, Seg* v, int &n)
{
	int m = 0, l, r;
	for (int d = 1; d*d <= ed; ++d) {
		l = (st+d-1)/d;
		r = ed/d;
		if (l <= r) {
			if (d < l || d > r)
				t[m++] = Seg(d, d);
			t[m++] = Seg(l, r);
		}
	}

	sort(t, t+m);
	t[m++] = Seg(inf, inf);
	
	l = t[0].first;
	r = t[0].second;
	n = 0;
	Rep (i, 1, m) {
		if (t[i].first > r) {
			v[n++] = Seg(l, r);
			l = t[i].first;
			r = t[i].second;
		} else
			r = max(r, t[i].second);
	}
}

int main()
{
	int n, s1, s2, w1, w2;
	scanf("%d", &n);
	while (n--) {
		scanf("%d%d%d%d", &s1, &s2, &w1, &w2);
		cal(s1, s2, s, ns);
		cal(w1, w2, w, nw);
		int ans = 1, i = 0, j = 0;
		while (i < ns) {
			while (j < nw && w[j].first <= s[i].second) {
				if (w[j].second >= s[i].first)
					ans = max(ans, min(s[i].second, w[j].second));
				++j;
			}
			--j;
			++i;
		}
		printf("%d\n", ans);
	}
	return 0;
}
```

POI果然非常喵.