---
title: "[bzoj 1057] [ZJOI2007]棋盘制作"
date: 2017-05-31 15:46:09
categories:
- bzoj
tags:
- 单调栈
---
n\*m的01矩阵, 分别求出相邻格子不同色的正方形, 矩形的最大面积. (n,m&le;2000)
<!--more-->
正方形的最大面积, DP一下就好. 矩形呢? 想采用类似的思路, 但是搞不定, 因为长和宽相互制约.

瞥了一眼吉司机的一句话题解......"直接DP就好了"??!

又想了一下. 既然吉司机这么说, 那么本题不需要特殊的奇技淫巧......

联想起之前做过的[[poj 2559] Largest Rectangle in a Histogram](http://blog.csdn.net/ruoruo_cheng/article/details/53102481), 发现只要特判一下相邻两格同色的情况, 本题就做完了.

为什么非要去瞥一眼题解才产生这个联想......TAT

换一个角度看. "相邻格子不同色", 实际上是说, 区域内, 行号+列号=奇数的格子是一种颜色, 行号+列号=偶数的格子是另一种颜色. 那么, 把行号+列号=偶数的所有格子的颜色取反, 本题就转化为同色的最大正方形/矩形. 或许经过这步转化后更有助于联想, 虽然它不是必要的.

再叙述一遍.

依次考虑正方形/矩形以第i=1,2,...,n行为下边界. 递推求出h[j]=第i行第j列向上延伸的最远距离. 接下来, 使用求直方图中最大子矩形的方法: 从左往右扫描, 单调栈维护一些二元组(x,h[j]) (h[j]单增), 表示从位置j, 以h[j]为高度, 最多向左延伸至x; 对于每个二元组(x,h[j]), 当扫描到第一个j'&gt;j且h[j']&le;h[j], 用长j'-x, 高h[j]来更新答案. 长和宽直接相乘得到极大矩形面积, 取min再平方得到极大正方形面积.

Por una Cabeza.

```cpp
#define x first
#define y second

typedef pair<int, int> P;

const int N = 2000;
int n, m, rec, sqr, a[N+1][N+2], h[N+2];

inline void update(int a, int b)
{
	int c = min(a, b);
	rec = max(rec, a*b);
	sqr = max(sqr, c*c);
}

void process(int a[])
{
	stack<P> S;
	P t;

	rep (i, 1, m+2) {
		int x = i;

		while (!S.empty() && (t = S.top(), a[i] == a[i-1] || t.y >= h[i])) {
			S.pop();
			update(i-t.x, t.y);
			if (a[i] != a[i-1]) x = t.x;
		}

		S.push(P(x, h[i]));
	}
}

int main()
{
	scanf("%d%d", &n, &m);
	rep (i, 1, n+1) {
		rep (j, 1, m+1) scanf("%d", &a[i][j]);
		a[i][m+1] = -1;
	}

	fill_n(h+1, m, 1);
	process(a[1]);

	rep (i, 2, n+1) {
		rep (j, 1, m+1)
			h[j] = a[i][j] == a[i-1][j] ? 1 : h[j] + 1;
		process(a[i]);
	}

	printf("%d\n%d\n", sqr, rec);

	return 0;
}
```