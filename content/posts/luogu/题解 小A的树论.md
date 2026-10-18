---
title: "题解 小A的树论"
date: 2023-03-04T14:55:30+08:00
draft: false
author: "Jsxts"
tags: ["个人记录"]
slug: "题解-小a的树论"
original: "https://www.luogu.com.cn/article/cgfwskjh"
math: true
---
## XSY 1019 小A的树论

考虑最难的操作移动子树，在 dfn 或者括号序上就可以看成将区间平移，这样就可以用 FHQ 把一段区间分裂出来再合并，由于需要快速找到 $x$ 子树代表的区间，可以用括号序，这样直接访问两个 $x$ 出现之间的区间就是 $x$ 的子树，然后剩下的随便维护就好了。

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;
const int inf = 2e9;
int read() {
	int s = 0,f = 1;char ch = getchar();
	while (!isdigit(ch)) f = ch == '-' ? -1 : 1, ch = getchar();
	while (isdigit(ch)) s = s * 10 + ch - '0', ch = getchar();
	return s*f;
}
void write(int x) {
	if (x < 0) putchar('-'), x = -x;
	if (x > 9) write(x / 10);
	putchar(x % 10 + 48);
}
int root,cnt,r1,r2,r3,dfn[100010],low[100010];
int a[100010];
struct edge {
	int v,nxt;
}ed[200010];
int head[100010],cn;
void add(int u,int v) {
	ed[++cn] = {v,head[u]};
	head[u] = cn;
}
struct FHQ_Treap {
	struct node {
		int l,r,sz,rnd,fa,cnt;
		ll v,s,tag;
	}t[200010];
	#define ls t[x].l
	#define rs t[x].r
	void pushup(int x) {
		t[x].s = t[ls].s + t[rs].s + t[x].v;
		t[x].sz = t[ls].sz + t[rs].sz + 1;
		t[x].cnt = t[ls].cnt + t[rs].cnt + 1;
	}
	void f(int x,ll s) {
		t[x].v += s, t[x].s += t[x].cnt * s;
		t[x].tag += s;
	}
	void pushdown(int x) {
		f(ls,t[x].tag), f(rs,t[x].tag);
		t[x].tag = 0;
	}
	void split(int x,int k,int &l,int &r) {
		if (!x) {
			l = r = 0;
			return;
		}
		pushdown(x);
		if (t[ls].sz < k) {
			l = x, split(rs,k - t[ls].sz - 1,rs,r);
			if (rs) t[rs].fa = x;
		}
		else {
			r = x, split(ls,k,l,ls);
			if (ls) t[ls].fa = x;
		}
		pushup(x);
	}
	int merge(int u,int v) {
		if (!u || !v) return u + v;
		pushdown(u), pushdown(v);
		if (t[u].rnd < t[v].rnd) {
			t[u].r = merge(t[u].r,v);
			if (t[u].r) t[t[u].r].fa = u;
			pushup(u);
			return u;
		}
		else {
			t[v].l = merge(u,t[v].l);
			if (t[v].l) t[t[v].l].fa = v;
			pushup(v);
			return v;
		}
	}
	int add_node(int x) {
		int y = ++cnt;
		t[y].l = t[y].r = t[y].fa = 0;
		t[y].v = t[y].s = x;
		t[y].sz = t[y].cnt = 1, t[y].tag = 0;
		t[y].rnd = rand();
		return y;
	}
	int rank(int x) {
		int res = t[ls].sz + 1;
		while (x != root) {
			if (x == t[t[x].fa].r) res += t[t[t[x].fa].l].sz + 1;
			x = t[x].fa;
		}
		return res;
	}
	ll query(int x) {
		split(root,rank(low[x]),r1,r2);
		split(r1,rank(dfn[x]) - 1,r1,r3);
		ll res = t[r3].s >> 1;
		root = merge(merge(r1,r3),r2);
		return res;
	}
	void modify(int x,int y) {
		split(root,rank(low[x]),r1,r2);
		split(r1,rank(dfn[x]) - 1,r1,r3);
		f(r3,y);
		root = merge(merge(r1,r3),r2);
	}
	void cut(int x,int y) {
		split(root,rank(low[x]),r1,r2);
		split(r1,rank(dfn[x]) - 1,r1,r3);
		root = merge(r1,r2);
		split(root,rank(dfn[y]),r1,r2);
		root = merge(merge(r1,r3),r2);
	}
}T;
void dfs(int x,int fa) {
	root = T.merge(root,dfn[x] = T.add_node(a[x]));
	for (int i = head[x];i;i = ed[i].nxt) {
		int v = ed[i].v;
		if (v != fa) dfs(v,x);
	}
	root = T.merge(root,low[x] = T.add_node(a[x]));
}
int main() {
	int n = read();
	int m = read();
	for (int i = 1;i <= n;i ++ ) a[i] = read();
	for (int i = 1;i < n;i ++ ) {
		int u = read(),v = read();
		add(u,v), add(v,u);
	}
	dfs(1,0);
	while (m -- ) {
		int op = read(),x = read();
		if (op == 1) printf("%lld\n",T.query(x));
		if (op == 2) T.modify(x,read());
		if (op == 3) T.cut(x,read());
	}
	return 0;
}
```
