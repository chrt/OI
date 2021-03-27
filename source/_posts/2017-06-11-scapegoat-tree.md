---
title: 替罪羊树
date: 2017-06-11 15:29:26
categories:
- 笔记
tags:
- 替罪羊树
---
同学问我替罪羊树和带插入的区间k小, 我说没有太大兴趣, 结果立了个flag......
本文不加证明地介绍了替罪羊树, 这种简洁高效, 不用旋转的重量平衡树.
link:
- cls的集训队论文 &lt;重量平衡树和后缀平衡树在信息学奥赛中的应用&gt;
- vfk的论文 &lt;对无旋转操作的平衡树的一些探究&gt;
- 发明人的论文 [Scapegoat Trees](http://www.akira.ruc.dk/~keld/teaching/algoritmedesign_f07/Artikler/03/Galperin93.pdf)
- neither_nor神犇对发明人论文的[翻译](http://blog.csdn.net/neither_nor/article/details/52347028)
<!--more-->

# 重量平衡树
每次操作影响的最大子树的大小是最坏, 或均摊, 或期望 $O(\lg n)$ 的平衡树叫重量平衡树.

替罪羊树是不采用旋转机制的重量平衡树. 插入均摊 $O(\lg n)$, 查询和删除 $O(\lg n)$.

Treap是采用旋转机制的重量平衡树. 插入, 查询, 删除期望 $O(\lg n)$. 插入的旋转次数是 $O(1)$ 的, 每次旋转重构子树, 期望复杂度 $O(\lg n)$. 删除直接重构子树, 时间同样是期望 $O(\lg n)$ 的.

# 两种平衡
$\frac 1 2 \le \alpha < 1$

如果一棵二叉树 $T$ 的高度满足 $h\le \log_{1/\alpha}(n)$, 则称 $T$ 是 $\alpha$ 高度平衡的. 记 $h_\alpha(n) = \lfloor \log_{1/\alpha}(n)\rfloor$

如果一个节点 $v$ 满足 $size(left(v))\le \alpha\cdot size(v), size(right(v))\le \alpha\cdot size(v)$, 则称 $v$ 是 $\alpha$ 大小平衡的. 如果 $T$ 的所有节点都 $\alpha$ 大小平衡, 则称 $T$ $\alpha$ 大小平衡.

大小平衡是高度平衡的充分非必要条件. 从最深的节点往上爬, 连续使用大小平衡的定义, $1\le \alpha^h n \Rightarrow h\le \log_{1/\alpha}(n)$. 反之不然.

# 替罪羊树
替罪羊树的每个节点, 除了左右儿子的指针, 不用记录其他信息. 方便起见, 可以记录 $size$.

## 查询
替罪羊树的查询和普通BST一样.

## 插入
先像普通BST一样插入. 维护 $maxn$ (无删除则省略). 如果深度 $h > h_\alpha(n)$, 回溯的时候找到第一个非 $\alpha$ 大小平衡的节点 $u$, 称 $u$ 为替罪羊节点. 将以 $u$ 为根的子树重建成1/2大小平衡的.

一定存在替罪羊节点, 因为非高度平衡可以推出非大小平衡.

重建操作可以在 $O(size(u))$ 的时间内完成. 简单地中序遍历一遍, 用 $O(n)$ 的空间即可. 也可以用 $O(\log n)$ (栈) 空间的*拍扁重建法*.

## 删除
先像普通BST一样删除. 如果 $n < \alpha \cdot maxn$, 重建整棵树, 置 $maxn$ 为 $n$.

# 性质
如果只有插入操作, 替罪羊树是 $\alpha$ 高度平衡的.

如果同时有插入,删除, 替罪羊树是非严格 $\alpha$ 高度平衡的, 即 $h \le h_\alpha(n)+1$.

$\alpha$ 可以根据需要来取. 通常取0.6~0.7.