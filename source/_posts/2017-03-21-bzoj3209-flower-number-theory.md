---
title: "[bzoj 3209] 花神的数论题"
date: 2017-03-21 14:54:51
categories:
- bzoj
tags:
- 数论
- 数位DP
- 排列组合
---
求 П(i的二进制表示中1的个数)[1&le;i&le;n] mod 10000007. (1&le;n&le;10^15)
<!--more-->
2^49 &lt; 10^15 &lt; 2^50. 数的二进制表示至多50位. 对于i=1,2,...,50, 统计范围内多少个数的二进制表示中1的个数为i, 快速幂, 再乘起来.

如果n是2的幂, 答案可以直接用组合数算出来. 这里n不一定是2的幂, 可以类似地做一个数位统计.

模m意义下的乘方运算, 如果底数与m互质, 指数可对$\varphi(m)$取模. 质因数分解: 10000007=941\*10627. $\varphi(10000007) = 9988440$, 且1~50与模互质.

唉? 又不会爆`long long`, 为什么要取模......n开到10^1000才正常嘛......

```cpp
#include <bits/stdc++.h>
#define For(i, a, b) for (int i = (a), i##_ = (b); i <= i##_; ++i)
#define Down(i, a, b) for (int i = (a), i##_ = (b); i >= i##_; --i)

using namespace std;
typedef long long ll;
const int MOD = 10000007, Phi = 9988440, L = 50;
int C[L + 1][L + 1];

void init()
{
	For (i, 0, L) {
		C[i][0] = 1;
		For (j, 1, i) {
			C[i][j] = C[i-1][j] + C[i-1][j-1];
			C[i][j] -= C[i][j] >= Phi ? Phi : 0;
		}
	}
}

ll fpm(ll x, ll n)
{
	ll y = 1;
	for (; n; n >>= 1, (x *= x) %= MOD)
		if (n & 1)
			(y *= x) %= MOD;
	return y;
}

ll solve(ll n)
{
	static ll cnt[L + 1];
	ll y = 0;
	Down (i, L-1, 0)
		if (n & (1LL<<i)) {
			For (j, 0, i) {
				cnt[j + y] += C[i][j];
				cnt[j + y] -= cnt[j + y] >= Phi ? Phi : 0;
			}
			++y;
		}
	ll ans = 1;
	For (i, 2, L)
		if (cnt[i])
			(ans *= fpm(i, cnt[i])) %= MOD;
	return ans;
}
	
int main()
{
	init();
	ll n;
	scanf("%lld", &n);
	printf("%lld\n", solve(n+1));
	return 0;
}
```
