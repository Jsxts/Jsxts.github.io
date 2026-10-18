---
title: "题解 P9990 [Ynoi Easy Round 2023] TEST_90"
date: 2023-12-26T09:00:11+08:00
draft: false
author: "Jsxts"
tags: ["题解"]
slug: "题解-p9990-ynoi-easy-round-2023-test-90"
original: "https://www.luogu.com.cn/article/3tvnc1yh"
math: true
---
容易将题目转化为区间将 $0$ 变 $1$，$1$ 变 $0$，查询区间历史和。

直接用[线段树维护历史和](https://www.luogu.com.cn/blog/yangsonghan/xian-duan-shu-wei-hu-li-shi-hu)的套路来推 $tag$：

$$hs+=\sum_{i=1}^m[q_i=upd][c_i=1]\times len+s\times(\sum_{i=1}^m[q_i=upd][c_i=0]-\sum_{i=1}^m[q_i=upd][c_i=1])$$

其中 $c_i$ 表示前缀取反的次数模 $2$。

那么考虑维护上述出现的量 $c_0,c_1$ 表示 $\sum_{i=1}^m[q_i=upd][c_i=0/1]$，以及区间被异或 $1$ 次数 $tag$。考虑合并两个队列：

$$c_0=\sum_{i=1}^{m_1+m_2}[q3_i=upd][c_i=0]$$
$$=lc_0+\sum_{i=1}^{m_2}[q2_i=upd][c2_i=tag_l]$$

已经可以维护，$c_1$ 同理。

代码：

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;
const int inf = 1e9;
const ll INF = 1e15;
const int N = 1e6;
inline int read() {
	int s = 0,f = 1;char ch = getchar();
	while (!isdigit(ch)) f = ch == '-' ? -1 : 1, ch = getchar();
	while (isdigit(ch)) s = (s << 3) + (s << 1) + ch - '0', ch = getchar();
	return s*f;
}
int n,a[N + 10],m,pre[N + 10];
vector<pair<int,int> > vc[N + 10];
struct Seg {
	int l,r,c0,c1,s,len,tag;
	ll hs;
}t[N * 4 + 10];
void build(int x,int l,int r) {
	t[x].l = l, t[x].r = r, t[x].len = r - l + 1;
	if (l == r) return;
	int mid = l + r >> 1;
	build(x<<1,l,mid), build(x<<1|1,mid + 1,r);
}
void pushup(int x) {
	t[x].s = t[x<<1].s + t[x<<1|1].s;
}
void f(int x,Seg s) {
	t[x].hs += 1ll * s.c1 * t[x].len + 1ll * t[x].s * (s.c0 - s.c1);
	t[x].c0 += t[x].tag ? s.c1 : s.c0;
	t[x].c1 += t[x].tag ? s.c0 : s.c1;
	t[x].tag ^= s.tag;
	t[x].s = s.tag ? t[x].len - t[x].s : t[x].s;
}
void pushdown(int x) {
	f(x<<1,t[x]), f(x<<1|1,t[x]);
	t[x].c0 = t[x].c1 = t[x].tag = 0;
}
void modify(int x,int l,int r) {
	if (t[x].l >= l && r >= t[x].r) {
		f(x,{0,0,0,0,0,0,1,0});
		return;
	}
	int mid = t[x].l + t[x].r >> 1;
	pushdown(x);
	if (mid >= l) modify(x<<1,l,r);
	if (mid < r) modify(x<<1|1,l,r);
	pushup(x);
}
void upd(int x) {
	t[x].c0 += !t[x].tag, t[x].c1 += t[x].tag;
	t[x].hs += t[x].s;
}
ll query(int x,int l,int r) {
	if (t[x].l >= l && r >= t[x].r) return t[x].hs;
	int mid = t[x].l + t[x].r >> 1;
	pushdown(x);
	ll res = 0;
	if (mid >= l) res = query(x<<1,l,r);
	if (mid < r) res += query(x<<1|1,l,r);
	return res;
}
ll ans[N + 10];
int main() {
	n = read();
	for (int i = 1;i <= n;i ++ ) a[i] = read();
	m = read();
	build(1,1,n);
	for (int i = 1;i <= m;i ++ ) {
		int l = read(),r = read();
		vc[r].push_back({l,i});
	}
	for (int i = 1;i <= n;i ++ ) {
		modify(1,pre[a[i]] + 1,i);
		pre[a[i]] = i;
		upd(1);
		for (auto v : vc[i]) ans[v.second] = query(1,v.first,i);
	}
	for (int i = 1;i <= m;i ++ ) printf("%lld\n",ans[i]);
	return 0;
}
```
