---
title: "题解 P9992 [Ynoi Easy Round 2024] TEST_130"
date: 2023-12-28T16:28:37+08:00
draft: false
author: "Jsxts"
tags: ["题解"]
slug: "题解-p9992-ynoi-easy-round-2024-test-130"
original: "https://www.luogu.com.cn/article/ipaz9fac"
math: true
---
考虑无脑一点的思路，设 $f_{x,j}$ 表示 $x$ 子树内当 $d=j$ 时的答案。那么 $x$ 对每个 $f_{x,j}$ 有 $j+1$ 的贡献，然后合并子树就是令 $f_{x,j}\leftarrow f_{x,j}+f_{v,j-1}$。

那么这个 dp 第二维类似深度，可以用长剖优化。一开始的 $x$ 的贡献可以看成区间加等差数列，直接用线段树维护 dp 数组即可。可能有点卡常。

代码：

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;
const int inf = 1e9;
const ll INF = 1e15;
const int N = 1e6;
namespace IO{//by cyffff
    int len=0;
    char ibuf[(1<<21)+1],*iS,*iT,out[(1<<23)+1];
    #if ONLINE_JUDGE
    #define gh() (iS==iT?iT=(iS=ibuf)+fread(ibuf,1,(1<<21)+1,stdin),(iS==iT?EOF:*iS++):*iS++)
    #else
    #define gh() getchar()
    #endif
    #define reg register
    inline int read(){
        reg char ch=gh();
        reg int x=0;
        while(ch<'0'||ch>'9') ch=gh();
        while(ch>='0'&&ch<='9') x=x*10+(ch^48),ch=gh();
        return x;
    }
    inline void putc(char ch){
        out[len++]=ch;
    }
    template<class T>
    inline void write(T x){
        if(x<0)putc('-'),x=-x;
        if(x>9)write(x/10);
        out[len++]=x%10+48;
    }
    inline void flush(){
        fwrite(out,1,len,stdout);
        len=0;
    }
    inline char getc(){
        char ch=gh();
        while(ch<'A'||ch>'Z') ch=gh();
        return ch;
    }
}
using IO::read;
using IO::write;
using IO::flush;
using IO::getc;
using IO::putc;
int n,m,head[N + 10],cnt,to[N + 10];
vector<pair<int,int> > q[N + 10];
struct edge {
	int v,nxt;
}ed[N * 2 + 10];
void add(int u,int v) {
	ed[++cnt] = {v,head[u]};
	head[u] = cnt;
}
int son[N + 10],mxd[N + 10],dep[N + 10],id[N + 10],tot;
void dfs1(int x) {
	mxd[x] = 1;
	for (int i = head[x];i;i = ed[i].nxt) {
		int v = ed[i].v;
		dep[v] = dep[x] + 1;
		dfs1(v);
		if (mxd[son[x]] < mxd[v]) son[x] = v, mxd[x] = mxd[v] + 1;
	}
}
void dfs2(int x) {
	id[x] = ++tot;
	if (son[x]) dfs2(son[x]);
	for (int i = head[x];i;i = ed[i].nxt) {
		int v = ed[i].v;
		if (v != son[x]) dfs2(v);
	}
}
ll ans[N + 10];
struct Seg {
	int l,r,k;
	ll b;
}t[N * 4 + 10];
#define ls (x<<1)
#define rs (x<<1|1)
void build(int x,int l,int r) {
	t[x].l = l, t[x].r = r;
	if (l == r) {
		to[t[x].l] = x;
		return;
	}
	int mid = l + r >> 1;
	build(x<<1,l,mid), build(x<<1|1,mid + 1,r);
}
inline void f(int x,ll k,ll b) {
	t[x].k += k;
	t[x].b += b;
}
inline void pushdown(int x) {
	f(ls,t[x].k,t[x].b), f(rs,t[x].k,t[x].b);
	t[x].k = t[x].b = 0;
}
void modify(int x,int l,int r,int k,ll b) {
	if (t[x].l >= l && r >= t[x].r) {
		t[x].k += k, t[x].b += b;
		return;
	}
	if (t[x].k || t[x].b) pushdown(x);
	int mid = t[x].l + t[x].r >> 1;
	if (mid >= l) modify(ls,l,r,k,b);
	if (mid < r) modify(rs,l,r,k,b);
}
void modify2(int x,int l,int r,ll b) {
	if (t[x].l >= l && r >= t[x].r) {
		t[x].b += b;
		return;
	}
	if (t[x].k || t[x].b) pushdown(x);
	int mid = t[x].l + t[x].r >> 1;
	if (mid >= l) modify2(ls,l,r,b);
	if (mid < r) modify2(rs,l,r,b);
}
ll query(int x,int k) {
	if (t[x].l == t[x].r) return 1ll * t[x].k * t[x].l + t[x].b;
	int mid = t[x].l + t[x].r >> 1;
	if (t[x].k || t[x].b) pushdown(x);
	if (mid >= k) return query(ls,k);
	return query(rs,k);
}
void dfs(int x) {
	if (son[x]) dfs(son[x]);
	modify(1,id[x],id[x] + mxd[x] - 1,1,1 - id[x]);
	for (int i = head[x];i;i = ed[i].nxt) {
		int v = ed[i].v;
		if (v == son[x]) continue;
		dfs(v);
		for (int j = id[v],k = id[x] + 1;j <= id[v] + mxd[v] - 1;k ++, j ++ ) {
			if (j == id[v] + mxd[v] - 1) modify2(1,k,id[x] + mxd[x] - 1,query(1,j));
			else t[to[k]].b += query(1,j);
		}
	}
	if (!son[x])
	for (auto v : q[x]) ans[v.second] = 1;
	else
	for (auto v : q[x]) ans[v.second] = query(1,id[x] + min(mxd[x] - 1,v.first));
}
int main() {
	n = read(), m = read();
	for (int i = 2;i <= n;i ++ ) add(read(),i);
	build(1,1,n);
	for (int i = 1;i <= m;i ++ ) {
		int x = read(),d = read();
		q[x].push_back({d,i});
	}
	dfs1(1), dfs2(1);
	dfs(1);
	for (int i = 1;i <= m;i ++ ) write(ans[i]), putc('\n');
	flush();
	return 0;
}
```
