---
title: "一个神秘的 DP 方法"
date: 2024-02-26T15:47:33+08:00
draft: false
author: "Jsxts"
tags: ["个人记录"]
slug: "一个神秘的-dp-方法"
original: "https://www.luogu.com.cn/article/hm01unz5"
---
在一些要往序列中插数来算方案数的 DP 题里，如果在处理插入两个数中间时的贡献较困难时（比如之前它们两个相邻有贡献，但是现在的改变使得它们不在相邻），可以考虑将填过的数钦定成若干连续段，这样只需要处理几种情况：

- 新增一个连续段。
- 连接两个连续段。
- 放到某个连续段的开头或结尾。

有可能会简化 dp 转移。

例：[ABC313Ex] Group Photo
