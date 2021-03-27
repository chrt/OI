---
title: "[Usaco2015 Feb]Censoring"
date: 2017-07-07 10:49:08
categories:
- USACO
tags:
- KMP
---
给出 S,T 两串, 构造一个新串 U. 往 U 的尾部依次添加 S 的字符, 若 U 的后缀为 T, 删掉这个后缀继续流程. (|S|,|T| &lt; 10^6, 保证每次删除后 U 不会为空)
<!--more-->
先开始想令 S' = T + S, 发现并不好做. 又想到一个哈希的做法. 为了避免除法, 可以用一个栈记录所有哈希值.

紧接着, 发现 KMP 就是做这件事的 - 在 S 中匹配 T. 那么, 用一个栈记录 S 的每个前缀对应的指针, 匹配后弹栈即可.

```cpp
const int N = 1e6 + 2;

int n, m, top, next[N] = {-1};
char s[N], t[N], r[N];
stack<int> S;

void build()
{
	int j = 0;
	rep (i, 2, n+1)
	{
		while (j >= 0 && s[i] != s[j+1]) j = next[j];
		next[i] = ++j;
	}
}

void work()
{
	int j = 0;
	S.push(0);
	rep (i, 1, m+1)
	{
		while (j >= 0 && t[i] != s[j+1]) j = next[j];
		if (++j == n)
		{
			rep (k, 0, n-1) --top, S.pop();
			j = S.top();
		}
		else
		{
			r[top++] = t[i];
			S.push(j);
		}
	}
}

int main()
{
	scanf("%s%s", t+1, s+1);
	m = strlen(t+1), n = strlen(s+1);
	build();
	work();
	rep (i, 0, top) putchar(r[i]);
	putchar('\n');
	return 0;
}
```
