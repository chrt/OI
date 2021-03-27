---
title: "[Usaco2007 Feb]Cow Sorting牛排序"
date: 2017-07-14 16:50:27
categories:
- USACO
tags:
- 置换
- 贪心
---
交换 x,y 两个数的代价为 x+y. 给一个长度为 n 的序列, 用交换操作排序, 求最小代价. (n &le; 10^4, 每个数是 [1,10^5] 中两两不同的整数)
<!--more-->
a 要换到 b 的位置, 就从 a 向 b 连边. 我们得到了一个置换, 也就是一个个分离的环.

然后, 我做了一个错误的猜想: 每次操作至少得有一个数复原, 那么只需求出每个环里相邻两数之和的最小值......

学习了一下[黄学长的题解](http://hzwer.com/3905.html).

用交换操作使一个环里的数复位, 有一个更优美的方法: 选一个数`x`, 倒着滚一圈. 代价为`S + (l-2) x`. `l`是环的大小.

也可以把一个环外的数`y`拉到环内, 倒着滚一圈, 再换回去. 代价为`S + x + (l+1) y`.

显然, `x`取环内的最小值, `y`取整个序列的最小值. 使这个环复位的最小代价为`S + min((l-2)x, x + (l+1)y)`.

这套方案简洁明了, 且不劣于前面的错误猜想导出的方案, 有理由相信它是正确的. 没有找到证明, 暂时也没想到怎么证......

```cpp
const int N = 1e4, M = 1e5 + 1, inf = 2e9;

int ans, Min = inf, a[N], b[N], p[M], t[N];
bool v[N];

void visit(int y)
{
	int mn = inf, x = y, l = 0;
	do
	{
		v[x] = true;
		mn = min(mn, a[x]);
		++l;
		x = t[x];
	}
	while (x != y);
	ans += min((l-2)*mn, mn + (l+1)*Min);
}

int main()
{
	int n;
	scanf("%d", &n);
	rep (i, 0, n)
	{
		scanf("%d", a+i);
		Min = min(Min, a[i]);
		ans += a[i];
		p[b[i] = a[i]] = i;
	}
	sort(b, b+n);
	rep (i, 0, n)
		t[p[b[i]]] = i;
	rep (i, 0, n)
		if (!v[i])
			visit(i);
	printf("%d\n", ans);
	return 0;
}
```
