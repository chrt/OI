---
title: "[APIO 2014] Palindromes"
date: 2017-05-01 11:51:09
categories:
- APIO
tags:
- Manacher
- 后缀自动机
- 倍增
---
给一个小写拉丁字母组成的字符串s, 求所有回文子串长度\*出现次数的最大值. (1&le;|s|&le;3\*10^5)
<!--more-->
马拉车算法可以求出每个地方的最长回文半径. 一些较短的回文串会伴随一个长回文串出现. 建出后缀树, 在树上对以每个位置为中心的最长回文串的半边打标记 (如果不存在恰好表示它的结点, 则标记打到以它为前缀的最高位置; 奇偶需分别处理), 则每个子串作为回文串的半边的出现次数可以用DFS求出.

需要在后缀树上找到待标记的位置. 构造后缀树的时候保存每个后缀对应的结点, 根据子串长度在树上倍增即可. 后缀树可以用后缀自动机构造. 谨防MLE.

学习了一下网上的题解......

回忆一下马拉车的过程 (驾~驾~). 为什么有时候可以跳过一些字符? 1. 这一段回文串曾经出现过. 2. [j-mx, j+mx]是回文串. 所以, 马拉车算法枚举了所有本质不同的回文子串 (可能重复, 但不遗漏).

标记什么的都不需要......

推论: 本质不同的回文子串只有O(n)个.

```cpp
const int SIGMA = 26, N = 3e5, M = 2*N+5, D = 20;
const char X = 'z'+1;
typedef long long ll;
typedef pair<int, int> ii;

struct Node {
	Node* trans[SIGMA], * link;
	int len;
} node[M], * root = node;

ll ans;
int ptr = 1, maxd, id[N], anc[M][D+1], f[2*N+1];
vector<int> adj[M], cnt[2][M];

void build(const char s[], int n)
{
	Node* last = root;
	
	per (i, n-1, 0) {
		int c = s[i] - 'a';
		Node* now = node + ptr, * p = last, * q, * nq;
		(last = now)->len = n-i;
		id[i] = ptr++;

		while (p && !p->trans[c]) {
			p->trans[c] = now;
			p = p->link;
		}

		if (!p) {
			now->link = root;
			continue;
		}

		q = p->trans[c];
		if (q->len == p->len + 1) {
			now->link = q;
		} else {
			now->link = nq = node + ptr++;
			*nq = *q;
			nq->len = p->len + 1;
			q->link = nq;
			while (p && p->trans[c] == q) {
				p->trans[c] = nq;
				p = p->link;
			}
		}
	}

	rep (i, 1, ptr)
		adj[node[i].link - node].push_back(i);
}

void dfs(int u)
{
	rep (i, 0, maxd)
		anc[u][i+1] = anc[anc[u][i]][i];
	rep (i, 0, adj[u].size()) {
		int v = adj[u][i];
		anc[v][0] = u;
		dfs(v);
	}
}

int query(int x, int l)
{
	per (i, maxd, 0) {
		int y = anc[x][i];
		if (node[y].len >= l)
			x = y;
	}
	return x;
}

ii dfs2(int u)
{
	ll t[2] = {0, 0};
	rep (i, 0, adj[u].size()) {
		ii p = dfs2(adj[u][i]);
		t[0] += p.first;
		t[1] += p.second;
	}
	rep (i, 0, 2) {
		ans = max(ans, t[i] * ((node[u].len - i)*2 + i));
		rep (j, 0, cnt[i][u].size())
			ans = max(ans, ++t[i] * ((cnt[i][u][j] - i)*2 + i));
	}
	return ii(t[0], t[1]);
}

void manacher(char s[], int n)
{
	int j = -1, mx = -1;
	rep (i, 0, n) {
		int k = i < mx ? min(mx - i, f[2*j-i]) + 1 : 1;
		while (i+k<n && i>=k && s[i+k]==s[i-k]) ++k;
		if ((f[i] = --k) + i > mx) {
			mx = f[i] + i;
			j = i;
		}
	}
}

int main()
{
	static char s[N+1], t[2*N+1];
	scanf("%s", s);
	int n = 0;
	while (s[n]) {
		t[2*n] = X;
		t[2*n+1] = s[n];
		++n;
	}
	t[2*n] = X;
	while ((1<<maxd) < n) ++maxd;
	manacher(t, 2*n+1);
	build(s, n);
	dfs(0);
	rep (i, 0, n) {
		rep (j, 0, 2) {
			int r = (f[i*2+j] + 1) / 2;
			if (r) {
				int tmp = query(id[i], r);
				cnt[j][tmp].push_back(r);
			}
		}
	}
	rep (i, 0, 2) rep (j, 1, ptr)
		sort(cnt[i][j].begin(), cnt[i][j].end(), greater<int>());
	dfs2(0);
	
	printf("%lld\n", ans);

	return 0;
}
```