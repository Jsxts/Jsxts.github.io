---
title: "题解 CF1045J Moonwalk challenge"
date: 2023-11-27T20:43:18+08:00
draft: false
author: "Jsxts"
tags: ["题解"]
slug: "题解-cf1045j-moonwalk-challenge"
original: "https://www.luogu.com.cn/article/1b1xtbop"
math: true
---
来个 ACAM 的。

首先将一条路径拆成在两条链上的和跨过 lca 的。跨过 lca 的较为简单，由于 $|s_i|\le 100$，直接提取出中间长度为 $O(|s_i|)$ 的一段做 kmp 匹配即可。

现在的问题是如何求一个从上往下的链 $u\to v$ 组成的字符串中 $s_k$ 的出现次数 $occ((u,v),s_k)$。考虑容斥成 $occ((1,v),s_k)-occ((1,u),s_k)$ 再减去左右端点跨过 $u$ 的方案数。这个和上面跨过 lca 的类似。

那么求一条到根链中 $s_k$ 的出现次数可以离线建出 $s$ 的 ACAM，然后 dfs 原树，在 ACAM 上匹配，将走到的节点权值 $+1$，查询就是求 $s_k$ 在 ACAM 上的结束节点的子树权值和，用树状数组维护。发现 $u\to lca$ 这一段是反过来走的，所以还要建出 $s$ 的反串 ACAM，用反串在上面也匹配即可。

然而有一个很严重的问题：空间达到了 $(\sum|s_k|)\times |\sum| \times 2=5.2\times 10^8$，完全开不下。于是我们将询问分批次处理，每 $B$ 个询问重新建树，取 $B=5000$ 即可通过。注意 vector 清空时不能只 clear，要将内存释放出去。

复杂度：$O(\frac{n^2}{B}\log n)$。

代码：

```cpp
#include <bits/stdc++.h>
#define x first
#define y second
using namespace std;
typedef long long ll;
const int inf = 1e9;
const ll INF = 1e15;
const int N = 1e5;
inline int read() {
	int s = 0,f = 1;char ch = getchar();
	while (!isdigit(ch)) f = ch == '-' ? -1 : 1, ch = getchar();
	while (isdigit(ch)) s = (s << 3) + (s << 1) + ch - '0', ch = getchar();
	return s*f;
}
int n,q,head[N + 10],cnt,ch[N + 10];
struct edge {
	int v,w,nxt;
}ed[N * 2 + 10];
void add(int u,int v,int w) {
	ed[++cnt] = {v,w,head[u]};
	head[u] = cnt;
}
int dfn[N + 10],tot,dep[N + 10],son[N + 10],sz[N + 10],fa[N + 10],top[N + 10],rnk[N + 10];
void dfs(int x,int f) {
	sz[x] = 1, fa[x] = f;
	for (int i = head[x];i;i = ed[i].nxt) {
		int v = ed[i].v;
		if (v == f) continue;
		dep[v] = dep[x] + 1;
		ch[v] = ed[i].w;
		dfs(v,x);
		sz[x] += sz[v];
		if (sz[son[x]] < sz[v]) son[x] = v;
	}
}
void dfs2(int x,int tp) {
	top[x] = tp, dfn[x] = ++tot, rnk[tot] = x;
	if (son[x]) dfs2(son[x],tp);
	for (int i = head[x];i;i = ed[i].nxt) {
		int v = ed[i].v;
		if (v == fa[x] || v == son[x]) continue;
		dfs2(v,v);
	} 
}
int LCA(int x,int y) {
	while (top[x] != top[y]) {
		if (dep[top[x]] < dep[top[y]]) swap(x,y);
		x = fa[top[x]];
	}
	return dep[x] < dep[y] ? x : y;
}
int kth(int x,int k) {
	if (k <= 0) return x;
	while (1) {
		if (k <= dep[x] - dep[top[x]]) return rnk[dfn[x] - k];
		else k -= dep[x] - dep[top[x]] + 1, x = fa[top[x]];
	}
	assert(0);
}
struct KMP {
	int p[N + 10];
	int solve(char *t,char *s) {//occur(t,s)
		int ls = strlen(s),lt = strlen(t);
		if (lt < ls) return 0;
		p[0] = -1;
		for (int i = 1,j = -1;i < ls;i ++ ) {
			while (~j && s[j + 1] != s[i]) j = p[j];
			if (s[j + 1] == s[i]) j ++;
			p[i] = j;
		}
		int res = 0;
		for (int i = 0,j = -1;i < lt;i ++ ) {
			while (~j && s[j + 1] != t[i]) j = p[j];
			if (s[j + 1] == t[i]) j ++;
			if (j == ls - 1) res ++ , j = p[j];
		}
		return res;
	}
}kmp;
int ans[N + 10];
const int B = 20;
const int M = N * 100 / B;
struct ACAM {
	int ch[M + 10][26],End[M + 10],tot;
	void ins(char *s,int id) {
		int l = strlen(s),u = 0;
		for (int i = 0;i < l;i ++ ) {
			int x = s[i] - 'a';
			if (!ch[u][x]) ch[u][x] = ++tot;
			u = ch[u][x];
		}
		End[id] = u;
	}
	int fail[M + 10],tt,dfn[M + 10],low[M + 10];
	int head[M + 10],cnt;
	vector<pair<int,int> > vc[N + 10];
	struct edge {
		int v,nxt;
	}ed[M + 10];
	void add(int u,int v) {
		ed[++cnt] = {v,head[u]};
		head[u] = cnt;
	}
	void dfs(int x) {
		dfn[x] = ++tt;
		for (int i = head[x];i;i = ed[i].nxt) dfs(ed[i].v);
		low[x] = tt;
	}
	int c[M + 10];
	void upd(int x,int s) {
		for (;x <= tot + 1;x += (x & -x)) c[x] += s;
	}
	int getsum(int x) {
		int res = 0;
		for (;x;x -= (x & -x)) res += c[x];
		return res;
	}
	int ask(int x) {
		return getsum(low[End[x]]) - getsum(dfn[End[x]] - 1);
	}
	void build() {
		queue<int> q;
		for (int i = 0;i < 26;i ++ )
			if (ch[0][i]) q.push(ch[0][i]), fail[ch[0][i]] = 0;
		while (!q.empty()) {
			int u = q.front();q.pop();
			for (int i = 0;i < 26;i ++ )
				if (ch[u][i]) fail[ch[u][i]] = ch[fail[u]][i], q.push(ch[u][i]);
				else ch[u][i] = ch[fail[u]][i];
		}
		for (int i = 1;i <= tot;i ++ ) add(fail[i],i);
		dfs(0);
	}
	void init() {
		memset(fail,0,sizeof(int)*(tot+1));
		for (int i = 0;i <= tot;i ++ )
			for (int j = 0;j < 26;j ++ ) ch[i][j] = 0;
		memset(head,0,sizeof(int)*(tot+1));
		tot = cnt = tt = 0;
	}
}T[2];
void dfs3(int x,int u0,int u1) {
	T[0].upd(T[0].dfn[u0],1), T[1].upd(T[1].dfn[u1],1);
	for (auto v : T[0].vc[x]) ans[v.x] += v.y * T[0].ask(v.x);
	for (auto v : T[1].vc[x]) ans[v.x] += v.y * T[1].ask(v.x);
	for (int i = head[x];i;i = ed[i].nxt) {
		int v = ed[i].v;
		if (v == fa[x]) continue;
		dfs3(v,T[0].ch[u0][ch[v]],T[1].ch[u1][ch[v]]);
	}
	T[0].upd(T[0].dfn[u0],-1), T[1].upd(T[1].dfn[u1],-1);
}
char s[N + 10],t[N + 10];
int main() {
	n = read();
	for (int i = 1;i < n;i ++ ) {
		int u = read(),v = read();
		char ch[2];scanf("%s",ch);
		add(u,v,ch[0] - 'a'), add(v,u,ch[0] - 'a');
	}
	dfs(1,0), dfs2(1,1);
	q = read();
	for (int i = 1;i <= B;i ++ ) {
		if (q <= (i - 1) * N / B) return 0;
		T[0].init(), T[1].init();
		for (int i = 1;i <= n;i ++ ) {
			T[1].vc[i].clear(), T[0].vc[i].clear();
			T[1].vc[i].shrink_to_fit(), T[0].vc[i].shrink_to_fit();
		}
		for (int j = 1;j <= min(N / B,q - (i - 1) * N / B);j ++ ) {
			int u = read(),v = read();
			scanf("%s",s);
			int l = strlen(s);
			T[0].ins(s,j);
			reverse(s,s + l);
			T[1].ins(s,j);
			int lca = LCA(u,v),tt = 0;
			T[1].vc[u].push_back({j,1});
			if (lca != 1) T[1].vc[lca].push_back({j,-1});
			T[0].vc[v].push_back({j,1});
			if (lca != 1) T[0].vc[lca].push_back({j,-1});
			if (l == 1) continue;
			int p = kth(u,dep[u] - dep[lca] - l + 1),tp = p,q;
			tt = 0;
			for (;tp != 1 && dep[lca] - dep[tp] < l - 1;tp = fa[tp]) t[tt++] = ch[tp] + 'a';
			reverse(t,t + tt);
			t[tt] = '\0';
			ans[j] -= kmp.solve(t,s);
			reverse(s,s + l);
			q = kth(v,dep[v] - dep[lca] - l + 1),tp = q;
			tt = 0;
			for (;tp != 1 && dep[lca] - dep[tp] < l - 1;tp = fa[tp]) t[tt++] = ch[tp] + 'a';
			reverse(t,t + tt);
			t[tt] = '\0';
			ans[j] -= kmp.solve(t,s);
			tt = 0;
			for (tp = p;tp != lca;tp = fa[tp]) t[tt++] = ch[tp] + 'a';
			int tmp = tt;
			for (tp = q;tp != lca;tp = fa[tp]) t[tt++] = ch[tp] + 'a';
			reverse(t + tmp,t + tt);
			t[tt] = '\0';
			ans[j] += kmp.solve(t,s);
		}
		T[0].build(), T[1].build();
		dfs3(1,0,0);
		for (int j = 1;j <= min(N / B,q - (i - 1) * N / B);j ++ ) printf("%d\n",ans[j]), ans[j] = 0;
	}
	return 0;
}
```
