---
title: "[bzoj 3620] 似乎在梦中见过的样子"
date: 2017-07-07 13:05:19
categories:
- bzoj
tags:
- Brute Force
- KMP
---
给一个字符串 s 和常数 k, 问有多少个子串可以拆分成 A+B+A, 其中 |A| &ge; k, |B| &ge; 1. (子串不同当且仅当位置不同; 如果一个子串有多个拆分, 视作同一个) (n &le; 15000, k &le; 100)
<!--more-->
用 [NOI 2014] 动物园 的做法, 枚举左端点, 对每个前缀 i 求出长度小于 ceil(i/2) 的最长 border, 我们就可以用 O(n^2) 的时间解决本题.

具体而言: 先做一遍 KMP 求出 next 数组, 然后再做一遍 - 根据先前求出的 next 数组跳转 (不作修改), 但是保持 border 的长度合法. 因为 2(j+1) + 1 &le; i + 1 => 2j + 1 &le; i.

但是这个时间复杂度有点怪啊......

一翻题解, 还真的都是 O(n^2) 暴力...... QAQ

值得注意的是: 本题来自*2014湖北省队互测week2*, 也就是说它先于 动物园 这题产生...... Orz

```cpp
const int N = 15000 + 2;
char s[N];
int k, next[N] = {-1};

int calc(char* s, int n)
{
	int j = 0, ans = 0;
	rep (i, 2, n+1)
	{
		while (j >= 0 && s[i] != s[j+1]) j = next[j];
		next[i] = ++j;
	}
	j = 0;
	rep (i, 2, n+1)
	{
		j = 2*(j+1) == i ? next[j] : j;
		while (j >= 0 && s[i] != s[j+1]) j = next[j];
		ans += ++j >= k;
	}
	return ans;
}

int main()
{
	scanf("%s%d", s+1, &k);
	int ans = 0, n = strlen(s+1);
	rep (i, 0, n-2*k)
		ans += calc(s+i, n-i);
	printf("%d\n", ans);
	return 0;
}
```