---
title: "题解 P7880 [Ynoi2006] rldcot"
date: 2023-07-19T21:23:54+08:00
draft: false
author: "Jsxts"
tags: ["题解"]
slug: "题解-p7880-ynoi2006-rldcot"
original: "https://www.luogu.com.cn/article/zrf5ln02"
math: true
---
前面的讨论和第一篇题解相同，都是通过 dsu on tree 求出所有的有效点对共 $O(n\log n)$ 对，之后再进行查询。这里是后面查询的另一种维护方式。

对于查询，将点对和询问放到二维平面上，设查询区间为 $[l,r]$，则我们要求的即为：$x\ge l,y\le r$ 的所有点 $(x,y)$ 的颜色数。

注意到询问矩阵是 2-side 的，考虑离线后将 $x$ 这一维从大到小扫描线，再维护 $y$ 坐标上的区间数颜色。因为第二维查询是前缀查询，故 $y$ 坐标越小的同颜色点越优，所以我们可以维护 $fir_i$ 表示 $i$ 这个颜色在扫过的矩形中 $y$ 坐标的最小值。

现在考虑新加入一个点，设其纵坐标为 $y$，颜色为 $c$。若 $y<fir_c$，则对 $[y,fir-1]$ 这个区间有 $1$ 的贡献，并将 $fir_c\to y$。查询就直接单点查询，这个可以用树状数组维护。

$O(n\log^2n+m\log n)$。

代码挺短的。

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;
const int inf = 2e9;
const int N = 1e5;
const int M = 5e5;
inline int read() {
	int s = 0,f = 1;char ch = getchar();
	while (!isdigit(ch)) f = ch == '-' ? -1 : 1, ch = getchar();
	while (isdigit(ch)) s = (s << 3) + (s << 1) + ch - '0', ch = getchar();
	return s*f;
}
int n,m,head[N + 10],cnt;
struct edge {
	int v,w,nxt;
}ed[N * 2 + 10];
void add(int u,int v,int w) {
	ed[++cnt] = {v,w,head[u]};
	head[u] = cnt;
}
int dep[N + 10],fa[N + 10],sz[N + 10],son[N + 10];
ll dis[N + 10],h[N + 10];
void dfs(int x,int f,int d) {
	dep[x] = d, fa[x] = f, sz[x] = 1;
	for (int i = head[x];i;i = ed[i].nxt) {
		int v = ed[i].v;
		if (v == f) continue;
		dis[v] = dis[x] + ed[i].w;
		dfs(v,x,d + 1);
		sz[x] += sz[v];
		if (sz[son[x]] < sz[v]) son[x] = v;
	}
}
int tot,tn;
set<int> S[N + 10];
struct node {
	int x,v;
};
vector<node> vc[N + 10];
void calc(int x,int fa,int rt) {
	if (S[rt].size()) {
		int l = *(--S[rt].lower_bound(x)),r = *S[rt].lower_bound(x);
		if (l)
		vc[l].push_back({x,(int)dis[rt]});
		if (r != inf)
		vc[x].push_back({r,(int)dis[rt]});
	}
	for (int i = head[x];i;i = ed[i].nxt) {
		int v = ed[i].v;
		if (v == fa) continue;
		calc(v,x,rt);
	}
}
void join(int x,int fa,int rt) {
	S[rt].insert(x);
	for (int i = head[x];i;i = ed[i].nxt) {
		int v = ed[i].v;
		if (v == fa) continue;
		join(v,x,rt);
	}
}
void dfs3(int x,int f,int fl) {
	for (int i = head[x];i;i = ed[i].nxt) {
		int v = ed[i].v;
		if (v == f || v == son[x]) continue;
		dfs3(v,x,0);
	}
	if (son[x]) dfs3(son[x],x,1);
	swap(S[x],S[son[x]]);
	vc[x].push_back({x,(int)dis[x]});
	S[x].insert(0), S[x].insert(inf);
	for (int i = head[x];i;i = ed[i].nxt) {
		int v = ed[i].v;
		if (v == f || v == son[x]) continue;
		calc(v,x,x);
		join(v,x,x);
	}
	S[x].insert(x);
	if (!fl) S[x].clear();
}
vector<pair<int,int> > vc2[N + 10];
int fir[N + 10],ans[M + 10];
int c[N + 10];
void upd(int x,int s) {
	for (;x <= n;x += (x & -x)) c[x] += s;
}
int getsum(int x) {
	int res = 0;
	for (;x;x -= (x & -x)) res += c[x];
	return res;
}
int main() {
	n = read(), m = read();
	for (int i = 1;i < n;i ++ ) {
		int u = read(),v = read(),w = read();
		add(u,v,w), add(v,u,w);
	}
	dfs(1,0,1);
	for (int i = 1;i <= n;i ++ ) h[i] = dis[i];
	sort(h + 1,h + n + 1);
	tn = unique(h + 1,h + n + 1) - h - 1;
	for (int i = 1;i <= n;i ++ ) dis[i] = lower_bound(h + 1,h + tn + 1,dis[i]) - h;
	dfs3(1,0,0);
	for (int i = 1;i <= tn;i ++ ) fir[i] = n + 1;
	for (int i = 1;i <= m;i ++ ) {
		int l = read(),r = read();
		vc2[l].push_back({r,i});
	}
	for (int i = n;i;i -- ) {
		for (auto v : vc[i]) {
			if (v.x < fir[v.v]) upd(v.x,1), upd(fir[v.v],-1), fir[v.v] = v.x;
		}
		for (auto v : vc2[i]) {
			ans[v.second] = getsum(v.first);
		}
	}
	for (int i = 1;i <= m;i ++ ) printf("%d\n",ans[i]);
	return 0;
}
```
