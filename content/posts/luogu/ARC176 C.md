---
title: "ARC176 C"
date: 2024-04-23T13:34:33+08:00
draft: false
author: "Jsxts"
tags: ["题解"]
slug: "arc176-c"
original: "https://www.luogu.com.cn/article/w0grqm9l"
math: true
---
写得最抽象的一篇题解。

首先对于一个点，若其有权值相同的出边，则 它的权值 $w$ 就已经知道了，进而与其连着权值不为 $w$ 的边对应的点的权值也知道了，而连着权值为 $w$ 的边的点 $u$ 则获得了一个限制：$p_u<w$，记 $mn_u$ 表示 $u$ 已经得到的限制为：$p_u\le mn_u$。上述过程可以用队列模拟出。其中也包含着一些简单的是否无解的特判。

接下来，对于相互不知道点权的点建边。对于每个连通块，对其边数进行讨论（此时每个连通块的边权肯定各不相同，否则也无解）：

- 边数 $\ge$ 点数：若 $>$ 显然无解，若 $=$ 的话，每个点权会恰好对应一个边权，但是容易发现一个结论：一个点的点权必须恰好等于其最小的出边，故容易证明此时没有解。

- 边数 $=$ 点数 $-1$：即其中恰好有一个点不为边中出现的边权（称其为关键点）。还是由上面的结论，我们发现：若存在 $u,v$，满足 $u$ 最小边指向 $v$ 且 $v$ 最小边指向 $u$，那么其中必有一个是关键点。而对于一棵树，这样的对总是存在的（考虑从某个点出发一直沿最小边走，最后一定会折返）。所以，我们找到一个这样的对，然后钦定其中某个是关键点，再 dfs 整个树看其它点是否合法（其它点的点权已经确定）。

不过，对于剩下的点的方案数还需要一些讨论。剩下的点，我们可以看成是：有些数是可选的，然后需要为某些点定一个权值，并且每个点有个形如 $p_i\le w$（记为 $mn2_i$） 的限制。这个只需要按 $mn2_i$ 从小到大考虑即可。

对于一个关键点对 $(u,v)$ 来说：能成为关键点还需要看 $mn_u,mn_v$ 的限制是否满足：$u$ 能是关键点首先要满足 $mn_v\ge w$，$v$ 同理（因为另一个被钦定成 $w$）。

- 若 $mn_u\ge w,mn_v\ge w$，则将方案数乘 $2$，并加入一个点满足 $p_i\le w$。

- 若只有一个满足（记为 $v$），则只加入一个点，限制为 $p_i\le mn_u$。

- 若都不满足则无解。

对于单点，则加入限制为 $p_i\le mn_i$ 的点。

```cpp
#include <bits/stdc++.h>
#define x first
#define y second
using namespace std;
typedef long long ll;
const int inf = 1e9;
const ll INF = 1e15;
const int N = 5e5;
inline int read() {
	int s = 0,f = 1;char ch = getchar();
	while (!isdigit(ch)) f = ch == '-' ? -1 : 1, ch = getchar();
	while (isdigit(ch)) s = (s << 3) + (s << 1) + ch - '0', ch = getchar();
	return s*f;
}
const int mod = 998244353;
int getmod(int x) {
	return x - (x >= mod) * mod;
}
int qpow(int a,int b) {
	int res = 1;
	while (b) {
		if (b & 1) res = 1ll * res * a % mod;
		b >>= 1, a = 1ll * a * a % mod;
	}
	return res;
}
int n,head[N + 10],cnt,m,val[N + 10],mn2[N + 10];
map<pair<int,int>,int> mp;
struct edge {
	int v,w,nxt;
}ed[N * 2 + 10];
void add(int u,int v,int w) {
	ed[++cnt] = {v,w,head[u]};
	head[u] = cnt;
}
vector<pair<int,int> > vc[N + 10];
void add2(int u,int v,int w) {
	vc[u].push_back({v,w});
}
int fa[N + 10],ct[N + 10];
int find(int x) {
	return fa[x] == x ? x : fa[x] = find(fa[x]);
}
void join(int x,int y) {
	x = find(x), y = find(y);
	if (x == y) return;
	fa[x] = y, ct[y] += ct[x];
}
int vis[N + 10],ct2[N + 10],fl,cv[N + 10],tp,ans = 1;
pair<int,int> mn[N + 10];
int t[N + 10],tag[N + 10];
void dfs2(int x,int fa) {
	for (auto i : vc[x]) {
		int v = i.x,w = i.y;
		if (v == fa) continue;
		if (mn[x].y == v && mn[v].y == x) {
			if (fl) puts("0"), exit(0);
			fl = 1;
			tp = x;
			int ct = (mn2[x] >= w) + (mn2[v] >= w);
			if (!ct) puts("0"), exit(0);
			else if (ct == 1) t[min(mn2[x],mn2[v])] ++, tp = mn2[x] < mn2[v] ? x : v;
			else ans = 1ll * ans * ct % mod, t[w - 1] ++;
		}
		dfs2(v,x);
	}
}
void dfs(int x,int fa) {
	for (auto i : vc[x]) {
		int v = i.x,w = i.y;
		if (v == fa) continue;
		val[v] = w;
		if (x != tp && max(val[x],val[v]) != val[v]) {
			puts("0"), exit(0);
		}
		dfs(v,x);
	}
}
int main() {
	n = read(),m = read();
	for (int i = 1;i <= n;i ++ ) mn2[i] = inf, fa[i] = i;
	for (int i = 1;i <= m;i ++ ) {
		int u = read(),v = read(),w = read();
		add(u,v,w), add(v,u,w);
		if (w == 1) puts("0"), exit(0);
		if (mp[{u,w}]) {
			if (val[u] && val[u] != w) {
				puts("0");
				exit(0);
			}
			else if (!val[u]) {
				val[u] = w;
				tag[u] = 1;
			}
		}
		else if (mp[{v,w}]) {
			if (val[v] && val[v] != w) {
				puts("0");
				exit(0);
			}
			else if (!val[v]) {
				val[v] = w;
				tag[v] = 1;
			}
		}
		else {
			mp[{u,w}] = v, mp[{v,w}] = u;
			if (cv[w]) puts("0"), exit(0);
		}
		cv[w] = 1;
	}
	queue<int> q;
	for (int i = 1;i <= n;i ++ ) if (tag[i]) q.push(i);
	while (!q.empty()) {
		int u = q.front();q.pop();
		for (int i = head[u];i;i = ed[i].nxt) {
			int v = ed[i].v;
			if (ed[i].w == val[u]) {
				mn2[v] = min(mn2[v],val[u] - 1);
				continue;
			}
			if (ed[i].w < val[u] || (val[v] && max(val[u],val[v]) != ed[i].w)) puts("0"), exit(0);
			if (!val[v]) {
				val[v] = ed[i].w;
				q.push(v);
			}
		}
	}
	for (int i = 1;i <= n;i ++ ) mn[i] = {inf,inf};
	for (int i = 1;i <= n;i ++ ) {
		if (val[i]) continue;
		for (int j = head[i];j;j = ed[j].nxt) {
			int v = ed[j].v;
			if (val[v]) continue;
			add2(i,v,ed[j].w);
			join(i,v);
			ct[find(i)] ++;
			mn[i] = min(mn[i],{ed[j].w,v});
		}
	}
	for (int i = 1;i <= n;i ++ ) ct2[find(i)] ++;
	for (int i = 1;i <= n;i ++ ) {
		if (find(i) != i || val[i]) continue;
		if (ct[i] / 2 == ct2[i]) {
			puts("0");
			exit(0);
		}
		if (ct2[i] == 1) {
			t[min(n,mn2[i])] ++;
			continue;
		}
		fl = 0, tp = 0;
		dfs2(i,0);
		dfs(tp,0);
	}
	if (t[0]) puts("0"), exit(0);
	int ct = 0;
	for (int i = 1;i <= n;i ++ ) {
		if (!cv[i]) ct ++;
		while (t[i] -- ) ans = 1ll * ans * ct % mod, ct --;
	}
	cout << ans;
	return 0;
}
```
