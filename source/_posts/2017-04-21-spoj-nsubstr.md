---
title: "[spoj NSUBSTR] Substrings"
date: 2017-04-21 21:37:43
categories:
- spoj
tags:
- 后缀自动机
---
一个由小写拉丁字母组成的字符串S, 对1&le;i&le;|S|, 回答长度为i的子串的出现次数的最大值F(i). (|S|&le;2.5\*10^5)
<!--more-->
建出后缀自动机, 每个状态的|endpos|即为出现次数, 用它来更新此状态对应的长度区间.

然而......

1. 本题的时限很紧, 每个测试点0.100s-0.149s. 线段树是用不成的.
2. |endpos|该怎么求呢?

解决方案:
1. 用|endpos(s)|更新F(len(s)), 再用F(i+1)更新F(i). 也就是说, 用|endpos(s)|直接更新[1, len(s)]. 一定会有更大的值去更新[1, shortest(s)), 所以这样是正确的.
2. 对于所有不是由分裂产生的结点v, 令cnt[v] = 1, 否则cnt[v] = 0. 在Parent树上累加子树的cnt值即得. 为什么呢? 累加前, cnt[v] = 1 当且仅当 v 代表一个前缀 (以某一位置结束的最长字符串); 累加相当于计算它对自己后缀的贡献.

```cpp
const int SIGMA = 26, N = 2.5e5;

struct Node {
	Node* ch[SIGMA], * link;
	int len, cnt;
} nodes[N*2], * root = nodes, * ptr = nodes + 1, * last = root;

void add(int c)
{
	Node* cur = ptr++, * p = last;
	cur->len = p->len + 1;
	cur->cnt = 1;
	last = cur;
	
	while (p && !p->ch[c]) {
		p->ch[c] = cur;
		p = p->link;
	}

	if (!p) {
		cur->link = root;
		return;
	}
	
	Node* q = p->ch[c];
	
	if (q->len == p->len + 1) {
		cur->link = p->ch[c];
	} else {
		Node* nq = ptr++;
		*nq = *q;
		nq->len = p->len + 1;
		nq->cnt = 0;
		q->link = cur->link = nq;

		while (p && p->ch[c] == q) {
			p->ch[c] = nq;
			p = p->link;
		}
	}
}

char s[N+1];
int ans[N+1];
vector<int> adj[2*N];

template<typename T>
inline void upmax(T& x, T v)
{
	x = max(x, v);
}

int dfs(int u)
{
	int sum = nodes[u].cnt;
	rep (i, 0, adj[u].size())
		sum += dfs(adj[u][i]);
	upmax(ans[nodes[u].len], sum);
	return sum;
}

int main()
{
	scanf("%s", s);
	int n = strlen(s);
	rep (i, 0, n)
		add(s[i] - 'a');
	for (int i = 1; nodes+i != ptr; ++i)
		if (nodes[i].link)
			adj[nodes[i].link - nodes].push_back(i);
	dfs(0);
	per (i, n, 2)
		upmax(ans[i-1], ans[i]);
	rep (i, 1, n+1)
		printf("%d\n", ans[i]);
	return 0;
}
```