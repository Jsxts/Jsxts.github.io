---
title: "题解 CF500F New Year Shopping"
date: 2023-12-10T15:05:46+08:00
draft: false
author: "Jsxts"
tags: ["题解"]
slug: "题解-cf500f-new-year-shopping"
original: "https://www.luogu.com.cn/article/gjpi377i"
math: true
---
区间背包，猫树分治板题。

考虑离线后将每个询问 $[ql,qr]$ 存到猫树对应的区间上（即最大的满足 $ql,qr$ 在 $mid$ 两侧的区间），然后进行分治，对于右侧处理出每个前缀的背包数组，然后向左扫得到另一个背包，将得到的两个背包进行合并即可。注意合并时是枚举其中一边取 $i$ 个，另一边取 $\le b-i$ 个，所以要对之前的背包数组取前缀 $\max$。

$l=r$ 的时候要特殊处理一下。

时间复杂度 $O((n+q+t)v\log t)$。

代码：

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;
const int N = 4e3;
const int M = 2e4;
const int T = 1e4;
const int inf = 1e9;
const ll INF = 1e18;
int read() {
	int s = 0,f = 1;char ch = getchar();
	while (!isdigit(ch)) f = ch == '-'? -1 : 1, ch = getchar();
	while(isdigit(ch)) s = s * 10 + ch - '0', ch = getchar();
	return s * f;
}
int n,p,q;
vector<pair<int,int> > vc[M + 10];
struct node {
	int id,w,l,r;
};
vector<node> qu[M + 10],qu2[M + 10];
int find(int l,int r,int ql,int qr) {
	if (l == r) return l;
	int mid = l + r >> 1;
	if (ql <= mid && mid < qr) return mid;
	else if (ql > mid) return find(mid + 1,r,ql,qr);
	else return find(l,mid,ql,qr);
}
int cmp(node x,node y) {
	return x.l > y.l;
}
int ans[M + 10],f[T + 10][N + 10],g[N + 10];
void solve(int l,int r) {
	if (l == r) return;
	int mid = l + r >> 1;
	solve(l,mid), solve(mid + 1,r);
	if (!qu[mid].size()) return;
	for (int i = mid + 1;i <= r;i ++ ) {
		for (int j = 0;j <= N;j ++ ) f[i - mid][j] = f[i - mid - 1][j];
		for (auto v : vc[i])
			for (int j = N;j >= v.second;j -- )
				f[i - mid][j] = max(f[i - mid][j],f[i - mid][j - v.second] + v.first);
	}
	for (int i = mid + 1;i <= r;i ++ ) 
		for (int j = 1;j <= N;j ++ ) f[i - mid][j] = max(f[i - mid][j - 1],f[i - mid][j]);
	sort(qu[mid].begin(),qu[mid].end(),cmp);
	int p = 0;
	memset(g,-0x3f,sizeof g);
	g[0] = 0;
	for (int i = mid;i >= l;i -- ) {
		for (auto v : vc[i])
			for (int j = N;j >= v.second;j -- )
				g[j] = max(g[j],g[j - v.second] + v.first);
		for (;p < qu[mid].size() && qu[mid][p].l == i;p ++ ) {
			node t = qu[mid][p];
			for (int j = 0;j <= t.w;j ++ )
				ans[t.id] = max(ans[t.id],g[j] + f[t.r - mid][t.w - j]);
		}
	}
}
int main() {
	n = read(), p = read();
	for (int i = 1;i <= n;i ++ ) {
		int c = read(),h = read(),t = read();
		vc[t].push_back({h,c});
	}
	q = read();
	for (int i = 1;i <= q;i ++ ) {
		int a = read(),b = read();
		if (max(a - p + 1,1) == min(a,T)) {
			qu2[min(a,T)].push_back({i,b,0,0});
			continue;
		}
		if (max(a - p + 1,1) > min(a,T)) continue;
		qu[find(1,T,max(a - p + 1,1),min(a,T))].push_back({i,b,max(a - p + 1,1),min(a,T)});
	}
	memset(f[0],-0x3f,sizeof f[0]);
	f[0][0] = 0;
	solve(1,T);
	for (int i = 1;i <= T;i ++ ) {
		memset(g,-0x3f,sizeof g);
		g[0] = 0;
		for (auto v : vc[i])
			for (int j = N;j >= v.second;j -- )
				g[j] = max(g[j],g[j - v.second] + v.first);
		for (auto v : qu2[i])
			for(int j = 0;j <= v.w;j ++ ) ans[v.id] = max(ans[v.id],g[j]);
	}
	for (int i = 1;i <= q;i ++ ) printf("%d\n",ans[i]);
	return 0;
}
```
