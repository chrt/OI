---
title: "[bzoj 1188] [HNOI2007]分裂游戏"
date: 2017-05-31 07:59:51
categories:
- bzoj
tags:
- 组合游戏
---
n个瓶子, 第i个瓶子中有p[i]颗豆子. 每次选择3个瓶子i&lt;j&le;k (要求瓶子i中至少有一颗豆子), 从i中取走一颗豆子, 在j,k中各放入一颗豆子. 两人轮流操作, 无法操作者获胜. 问: 先手是否有必胜策略? 如果有, 求第一步字典序最小的取法, 和第一步不同走法的数目. (t&le;10组数据, 1&lt;n&le;21, p[i]&le;10000)
<!--more-->
我浅薄的博弈知识告诉我, 可能需要把分裂游戏拆成一些独立的小游戏之和, 再运用SG定理. 但是, 每次选3个瓶子, 它们相互影响.

每次选3个瓶子, 但是只选一颗豆子啊......豆子之间是相互独立的. 它们的SG值的异或和即整个游戏的SG值.

设位于瓶子x的豆子的SG值为sg(x). 选择一颗豆子i, 它分裂成了两颗j,k. 再次运用SG定理. 这种转移的SG值是`sg(j) xor sg(k)`.

第一步的走法, 枚举即可. 判断是否转移到先手必败态. 注意, 瓶子i中至少得有一颗豆子.

妙哉!

换一个角度看, 世界可能就不同. - 小强

```cpp
const int N = 21;

int n, a[N], sg[N];

int get_sg(int x)
{
	static bool f[N*N+1];
	fill_n(f, n*n+1, false);
	rep (i, x+1, n) rep (j, i, n)
		if ((sg[i] ^ sg[j]) <= n*n)
			f[sg[i] ^ sg[j]] = true;
	rep (i, 0, n*n+1)
		if (!f[i]) return i;
	assert(0);
}

int main()
{
	int t;
	scanf("%d", &t);
	while (t--) {
		scanf("%d", &n);
		rep (i, 0, n) scanf("%d", a+i);
		
		sg[n-1] = 0;
		per (i, n-2, 0) sg[i] = get_sg(i);
		
		int sum = 0, cnt = 0;
		rep (i, 0, n) if (a[i] & 1) sum ^= sg[i];
		
		if (sum == 0) {
			puts("-1 -1 -1\n0");
			continue;
		}
		
		rep (i, 0, n) if (a[i])
			rep (j, i+1, n) rep (k, j, n)
				if (!(sum ^ sg[i] ^ sg[j] ^ sg[k]) && cnt++ == 0)
					printf("%d %d %d\n", i, j, k);
		
		printf("%d\n", cnt);
	}
	return 0;
}
```