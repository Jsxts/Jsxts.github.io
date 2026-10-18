---
title: "题解 Grid Xor"
date: 2023-09-28T16:30:49+08:00
draft: false
author: "Jsxts"
tags: ["题解"]
slug: "题解-grid-xor"
original: "https://www.luogu.com.cn/article/8v2z6e7f"
---
手玩的。

考虑将图黑白染色，那么黑白方法是一样的，只关注如何选出若干黑格使得所有白格的贡献次数为奇数。

然后发现：

![](/images/luogu/88v359ls.png)

如图，只选第一条直线的黄格能得到蓝格的异或和，选前两条直线的黄格能得到红格的异或和，以此类推。

然后另一个方向的就反过来。

然后白格对黑格的方法类似。

然后做完了。
