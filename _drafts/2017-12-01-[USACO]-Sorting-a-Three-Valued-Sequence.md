---
layout: post
title: '[USACO] Sorting a Three-Valued Sequence'
description: ''
category: USACO
tags: USACO
---



题目描述

给定一个序列（共N个元素），序列中的每个元素来自集合{1, 2, 3}。将这个序列的元素两两交换，将序列变换为非降序排列的序列。求最小的交换次数。

题目分析

题目来源**IOI96**。

自己没有好的思路，我是看题解做的，囧。

一开始想到的是通过BFS遍历所有序列两两交换的结果，然后check结果是否是非降序。不过这个复杂度有些高。写了一半放弃了。

后来想是不是可以通过归并排序来处理（但没有论证过）。写了下归并排序之后，发现结果不对，作罢了。

网上看了题解，发现自己没有充分理解“**每个元素来自于集合{1, 2, 3}**”。每个元素的就在1、2、3中换来换去，这显然是个线索！！自己居然忽略了。上面考虑的那两个方法对于任意范围的元素都适合，显然不符合IOI的出题思路。

序列的每个位置无非是1、2、3，排序前和排序后，如果对应位置上的元素不变，直观上后续也无需进行交换。因此我们交换那些对应位置上排序前后变化的元素即可。因此是一个贪心算法。

这里我们证明一下如下论断：

> 如果排序后位置$k$上的元素和排序前位置$k$上的元素相同，则在整个序列进行交换的整个过程中，位置$k$上元素不参与任何交换。

证明：

1. 假设序列为数组$a$, 排序后为数组$b$,  其中$a[k] == b[k], k \in[1, N)$ 

2. 如果序列$a$和序列$b$一致，则得证；

3. 如果序列$a$和序列$b$不一致，则至少存在$i、j$,满足：

   $ a[i] \not= b[i], a[j] \not= b[j]$, 同时$a[j] == b[i] , (i \not= j, i,j \in[0, N))$。

   这里我们假设：$a[i] ==x, b[i] == a[j] == y, (x\not=y,x, y \in\{1,2,3\} )$

4. 如果$a[i] == b[j]==x<span style="color: rgb(132, 33, 162); display: inline"></span><span style="color: rgb(132, 33, 162); display: inline"></span><span style="color: rgb(132, 33, 162); display: inline"></span><span style="color: rgb(132, 33, 162); display: inline"></span><span style=" display: inline"></span><span style=" display: inline"></span>$, 则交换$a[i]、a[j]$，得证；

5. 如果$a[i] \not= b[j]$，即$b[j] \not=x$;同时由于$b[j] \not=a[j]$,因此$b[j]\not=y$。所以$b[j]$的值必定为$\{1,2,3\}$移除$x$、$y$之后剩下的那个数值，设为$t$；

6. 由于$b[j]$不和

7. ​



