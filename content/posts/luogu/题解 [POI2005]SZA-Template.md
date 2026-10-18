---
title: "题解 [POI2005]SZA-Template"
date: 2023-05-24T18:25:06+08:00
draft: false
author: "Jsxts"
tags: ["个人记录"]
slug: "题解-poi2005-sza-template"
original: "https://www.luogu.com.cn/article/4j8v2mmj"
math: true
---
牢记 $nxt$ 数组定义：$nxt_i$ 为前缀 $i$ 的最长 border。

这题，我们发现对于一个前缀 $i$，所有以 $i$ 作为 border 的前缀的末尾都可以盖上印章，则就是要判断所有以 $i$ 作为 border 的前缀的位置能否覆盖整个串，这些位置对应的就是 $i$ 在失配树上的子树。所有可能成为答案的前缀在树上是一条从根到整串代表的点的路径上，其中子树大小递减。于是我们可以从根开始，统计完当前节点的答案后删掉不属于下一节点的点，可以用双向链表支持同时更新答案。时间复杂度 $O(n)$。
