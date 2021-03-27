---
title: 后缀自动机
date: 2017-04-21 11:13:37
categories:
- 笔记
tags:
- 后缀自动机
toc: true
---
后缀自动机是一个接受某字符串所有后缀的有限状态自动机. 把字符集大小看作常数, 它的状态数, 转移数, 和构造的时间复杂度均是线性的.

后缀自动机同时具有自动机和树的性质. (光同时具有波和粒子的性质~)
<!--more-->
# Link
## 自动机视角
- WJMZBMR的营员交流
- [后缀自动机: O(N)的构建及应用 translate by wmdcstdio](http://blog.csdn.net/wmdcstdio/article/details/44780707)

## 后缀树视角
- [后缀自动机与线性构造后缀树 by fanhq666](http://fanhq666.blog.163.com/blog/static/8194342620123352232937/)
- [线性构造后缀树&后缀自动机 by 张云昊](http://blog.163.com/ps_lm/blog/static/20790406120125883433110/)

后面我的描述是基于自动机视角的......后来发现从后缀树的角度看更简单. 可能是因为对树比对自动机更熟悉吧.

## 复杂度
- [震惊！SAM复杂度竟如此显然！by ZQC](https://zhuanlan.zhihu.com/p/25948077)

# 有限状态自动机
有限状态自动机的功能是识别字符串.

自动机是一个五元组 (字符集alpha, 状态集合state, 初始状态init, 结束状态集合end, 状态转移函数trans).

令trans(s, c)表示处在状态s, 读入字符c后, 转移到的状态.

令trans(str)表示从init开始, 读入字符串str后, 转移到的状态.

# 终点集合 (endpos / Right)
把字符串的所有后缀插到一棵Trie树中, 当然可以实现后缀自动机的功能, 但是太暴力了......

设Reg(s)表示从状态s开始可以识别的字符串, 不难发现这是一些后缀的集合. 对于两个状态u,v, 如果Reg(u) = Reg(v), 那么它俩可以合并.

记endpos(str)表示子串str在母串中所有结束位置的集合. 一个后缀仅由它的开始位置确定, 所以, Reg(u) = Reg(v)等价于, 对于所有满足trans(s1)=u, trans(s2)=v的s1,s2, 有endpos(s1)=endpos(s2).

称s1和s2终点集合等价, 当且仅当endpos(s1)=endpos(s2). 进行完所有合并之后, 非初始状态数=终点集合等价类个数.

以下不加区分地对字符串和状态应用endpos记号.

设endpos(s1)∩endpos(s2)=r, |s1|&lt;|s2|. 那么, s1是s2的后缀. 所有s2的出现, 必伴随着s1的出现 (反之不然). 由此, endpos(s2)⊆endpos(s1).

如果s1是s2的后缀, 可推出endpos(s2)⊆endpos(s1).

因此, 两个终点集合之间仅有两种关系: 包含与被包含 (一个是另一个的后缀), 交集为空 (其他).

# 后缀链接 (suffix link / Parent)
一个状态v对应一个终点集合. 一堆终点集合等价的字符串按长度从大到小排序, 则后面的字符串是前面字符串的后缀.

这些长度必定是一个连续的区间 (如果s1, s3之间缺了一个s2, 那么endpos(s1)⊆endpos(s2)⊆endpos(s3), 而endpos(s1)=endpos(s3)). 设这个区间为[minlen(v), len(v)].

考虑s1的一个后缀t, 它的长度为minlen(v)-1. 它不和s1终点集合等价, 而跟另一些长度小于minlen(v)的s1的后缀等价: [minlen(u), len(u)], len(u) = minlen(v)-1. 从v向u连一条辅助边, 称为后缀链接, 令link(v)=u. 如果某终点集合的minlen=1, 则向初始状态init连边.

后缀链接不等于转移函数, 它起辅助作用, 就像AC自动机里的失配边.

设link(v)=u, endpos(u)真包含endpos(v)并且, endpos(u)是满足这一条件的所有集合中, 规模最小的一个. 由此, 所有后缀链接形成一棵以init为根的树形结构.

# 转移函数
设母串为T, trans(u,c)=v. 任取一个满足trans(s)=u的字符串s, 则trans(s+c) = v, endpos(s+c) = { r+1 | r∈endpos(s), T[r+1]=c } 即v所对应的终点集合. 即便可能存在另一个trans(u',c)=v, 得到的结果总是一样的.

考虑u和它在后缀链接树 (Parent树) 上的某个祖先p, 则有endpos(trans(u,c))⊆endpos(trans(p,c)). 所以, 满足trans(p,c)=v的p是后缀链接树某条链上连续的一段.

# 构造
后缀自动机的构造算法是在线的, 增量式的. 我们一个一个地添加字符. 对于每个结点 (状态) 记录3个量: 转移函数trans, 后缀链接link, 最大长度len. 暂时不标记结束状态.

一开始, 只有代表空串的初始状态init.

设已经构造了T, |T|=L. 现在添加字符x, 变成Tx. T的所有后缀从长到短依次是: t1, t2, ..., tk, 则我们要向后缀自动机中添加后缀t1+x, t2+x, ..., tk+x.

首先, 新建结点cur是必须的. len(cur) = L+1.

t1=last, 即之前整个字符串对应的状态. 通过后缀链接可以依次找到所有ti (i>1).

令p=第一个trans(ti, x) != nil的ti. 对于p之前的状态, 设置trans(\*, x) = cur (如果不存在这样的p, 则对所有ti设置).

设trans(p, x) = q. 这意味着某个tj+x已经出现在T中, 它是Tx的后缀.

但是状态q除了tj+x, 还可能表示某个以tj+x为后缀的串: w+x, w不是ti中的一个且|w|>|tj|.

当len(q)>len(p)+1时上述情况发生, 这时, 需要将状态q分裂成q和q', 使q'的长度为区间[min(q), len(p)+1], q的长度为区间[len(p)+2, len(q)] (这里的min(q)和len(q)是分裂之前的). 这样就可以直接令link(cur)=q'.

还需要修改一些指向q的转移函数, 使它们指向q'. 只需要考虑trans(ti, x). 根据先前的讨论, 我们知道满足trans(ti, x) = q的ti是连续的一段.

当len(q)=len(p)+1时, 直接令link(cur)=q.

# 后缀树
Parent树是原串的反串的后缀树. 构造算法也可以抛开endpos集合, 从后缀树的角度理解.

# 线性
设n为字符串的长度.

## 状态数
由构造算法可知, 每添加一个字符, 结点数至多增加2.

n=0, 状态数为1. n=1, 状态数为2. n=2, 状态数为3. n>2, 状态数至多为3+2(n-2)=2n-1.

abbbb...可以达到这一上界.

## 转移数
### 映射
设f是有限集A到有限集B的映射.

如果对于任意 b∈B, 不存在多于1个 a∈A, 使得 f(a)=b, 则f是*单射*. 每个象至多有一个原象, 所以|A|≤|B|.

如果对于任意 b∈B, 存在唯一的 a∈A, 使得 f(a)=b, 则f是*双射*. 原象存在且唯一, 所以|A|=|B|.

所以我们得到了一个比较两个集合大小的方法~

在数学读物中看到, 由于能构造偶数集到整数集的一一映射, 所以这俩集合是等势的. 刚刚在知乎上看到[如何证明(0,1)和[0,1]等势](https://www.zhihu.com/question/28670882), 童年的一大困惑......QAQ

### 非树边 -> 后缀
后缀自动机的转移形成了一个DAG, 任取一棵生成树. 考虑一条非树边, 一定存在一个后缀, 使得这条边是路径上的第一条非树边, 令这条非树边和那个后缀对应 (如果有多个符合条件的后缀, 任取其一). 这是一个 非树边->后缀 的单射, 因而 转移数=|树边|+|非树边|≤2n-2+n=3n-2 (n>2).

这棵生成树是任取的. 取一棵使得某后缀的路径全为树边的树 (这总是可以办到的), 则我们证明了 转移数≤3n-3 (n>2).

abbb...bbbc可以达到这一上界.

## 时间复杂度
### Code
```cpp
void add(int c)
{
	Node* cur = ptr++, * p = last;
	cur->len = last->len + 1;
	
	while (p && !p->ch[c]) {
		p->ch[c] = cur;
		p = p->link;
	}

	last = cur;

	if (!p) {
		cur->link = root;
		return;
	}
	
	Node* q = p->ch[c];
	
	if (q->len == p->len + 1) {
		cur->link = q;
	} else {
		Node* nq = ptr++;
		*nq = *q;
		nq->len = p->len + 1;
		q->link = cur->link = nq;

		while (p && p->ch[c] == q) {
			p->ch[c] = nq;
			p = p->link;
		}
	}
}
```

### Proof
先看第一个`while`.

记`L(x) = x->link->len`.

关注`L(cur)`. 对于每一次`add`:
- `p->len`单减, 且循环体的执行次数不多于`p->len`的减少量.
- 循环体至少执行一次. 循环一次后, p->len = L(last); 最后一次循环后, p->len = L(cur) - 1.
- 循环体的执行次数 ≤ L(last) - L(cur) + 2.

记`cur[i]`为第i次添加时的`cur`.

对于所有`add`:
∑ 循环体的执行次数 ≤ L(cur[0]) - L(cur[1]) + L(cur[1]) - L(cur[2]) + ... + L(cur[n-1]) - L(cur[n]) + 2n = L(cur[0]) - L(cur[n]) + 2n ≤ 2n.

看来可以用势能分析的语言重述......令L(cur)为势函数, 每次均摊代价不超过 -ΔL + 2 + ΔL = 2.

再看第二个`while`.

记`F(x) = x->link->link->len`.
- 循环体至少执行一次. 循环一次后, p->len ≤ F(last); 最后一次循环*前*, p->len ≥ nq->link->len = F(cur).
- 循环体的执行次数 ≤ F(last) - F(cur) + 2.

于是, ∑ 循环体的执行次数 ≤ 2n.

非常愉快~