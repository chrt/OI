---
title: "[bzoj 2257] [Jsoi2009]瓶子和燃料"
date: 2017-03-22 20:45:39
categories:
- bzoj
tags:
- 数论
---
从n个已知容量Vi而无刻度的瓶子中选k个, 用以下动作装燃料:
- 把一个瓶子倒满
- 把一瓶燃料全部倒掉
- 将燃料从瓶a倒向瓶b, 直到b满或a空

要求最大化一组瓶子能装的燃料的最小正体积, 输出这个最小正体积. (1&le;n&le;1000, 1&le;Vi&le;10^9, Vi为整数)
<!--more-->
一组瓶子能装的燃料的最小正体积等于容量的最大公约数.

首先, 一组瓶子能量出的所有体积都是各瓶子容量的线性组合. 虽然不能量出所有线性组合, 但值不大于最大容量的非负线性组合都是可以量出的: 通通往最大瓶子里倒, 如果最大瓶子满了却没倒完, 必定还有负项; 倒出这些负项, 然后继续这个过程.

线性组合集中的最小正元素等于基底的最大公约数. 当只有两个基底时, 这就是裴蜀定理. 先说说裴蜀定理吧.

$$ax + by = c\text{ 有解 } \Leftrightarrow c \backslash \gcd(a, b)$$

充分性是显然的, 必要性得证一证. 扩展欧几里德算法的存在, 使必要性也变得显然了. 从 <算法导论> 上学来一个非构造性的证明:

设$s$是线性组合集$\left\\{ax+by\right\\}$的最小正元素, 则$a\bmod s$也是线性组合集中的元素, 由于$s$的最小性, $a\bmod s = 0$, 即$s\backslash a$. 同理, $s\backslash b$. 于是, $s \le \gcd(a,b)$. $\gcd(a,b)$整除$a,b$的所有线性组合, 所以又有$s \ge \gcd(a,b)$. 综合起来, $s = \gcd(a,b)$.

多于两个基底时, 上述非构造性证明依然成立. $s$和基底的最大公约数相互整除, 由于均是正数. 所以它们相等.

给个构造性证明:

设基底为$(x_1, x_2, \ldots, x_n)$. 对$i=2,3,\ldots,n$, 用扩展欧几里德算法求出$p_i \gcd(x_1,x_2,\ldots,x_{i-1}) + q_i x_i = \gcd(x_1,x_2,\ldots,x_i)$, 从前往后代入即可.

回到本题. 开头所说的结论得到证明, 考虑怎样求n个正整数的k元子集的最大公约数的最小值.

以为可以二分, 但某一解不可行不意味大于它的解也不可行 (还在想这数据范围怎么这么小). 又以为可以暴力枚举因子每次暴力统计, 然后TLE; 难道不是$O(n^{2.5})$ ? 还真的不是......枚举的范围是$O(\sqrt V)$而不是$O(\sqrt n)$.

暴力枚举因子然后排序. 时间复杂度$O(n\sqrt V + n\sqrt V\lg nV)$. 这个上界不紧, 因为10^9以内最大约数个数约为1.6\*10^3.

```cpp
#include <bits/stdc++.h>
#define Rep(i, a, b) for (int i = (a), i##_ = (b); i < i##_; ++i)
#define Down(i, a, b) for (int i = (a), i##_ = (b); i >= i##_; --i)
using namespace std;
const int N = 1000, S = 2000;
int d[N*S], top;

int main()
{
	int n, k;
	scanf("%d%d", &n, &k);
	Rep (i, 0, n) {
		int v;
		scanf("%d", &v);
		for (int j = 1; j*j <= v; ++j)
			if (v % j == 0) {
				d[top++] = j;
				if (j*j != v)
					d[top++] = v/j;
			}
	}
	sort(d, d+top);
	int cnt = 0;
	Down (i, top-1, 0) {
		if (++cnt >= k)
			return printf("%d\n", d[i]), 0;
		if (i && d[i-1] != d[i])
			cnt = 0;
	}
	return 0;
}
```