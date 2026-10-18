---
title: "题解 CF625E Frog Fights"
date: 2023-10-26T11:45:41+08:00
draft: false
author: "Jsxts"
tags: ["题解"]
slug: "题解-cf625e-frog-fights"
original: "https://www.luogu.com.cn/article/932mzhzu"
math: true
---
考虑维护目前局面最先相撞的一对，容易发现肯定是相邻的一对。

然后我们对于每个青蛙，维护以下值：

- 上次更新速度时的全局时间 $t$。

- 当前速度 $v$。

- 走过的总路程 $d$。

容易发现，若某个时刻 $d_x-d_y>dis(x,y)$ 代表 $x$ 把 $y$ 撞飞了。

所以我们容易计算出每个青蛙撞飞下一个青蛙的时间（若撞不到设为 $\infty$，注意这里指的是从一开始经过的时间），将它们放进一个堆里，每次取出最小的更新堆，若最小的为 $\infty$ 则结束。最后留在堆内的就是答案。

那么考虑如何计算撞飞时间以及如何更新堆。

先计算出 $d'_x,d'_y$ 为当前时间的 $d$，即 $d'_x=d_x+(now-t_x)\times v_x$。

若 $v_x\le v_y$，若 $x$ 先撞并且一步能到先判掉，否则就撞不到。

否则若 $y$ 先走，由 $d'_x+tv_x-(d'_y+tv_y)>dis(x,y)$ 得 $t>\frac{d'_y-d'_x+dis(x,y)}{v_x-v_y}$，即 $t=\lceil\frac{d'_y-d'_x+dis(x,y)}{v_x-v_y}\rceil$。当 $x$ 先走时类似，得 $t=\lceil\frac{d'_y-d'_x+dis(x,y)-v_x}{v_x-v_y}\rceil+1$。

对于更新堆，我们维护一个循环双向链表，即要把 $(pre_x,x),(x,nxt_x)$ 更新。但是堆不支持删除，所以使用延迟删除，标记每个数在堆中的最后加入时刻。注意还有一种特殊情况是同时撞飞很多青蛙，在撞飞 $nxt_x$ 后接着往后判断即可。

$O(n\log n)$。
```cpp
#include <bits/stdc++.h>
#define int long long
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
int n,m,tot,vis[N + 10];
struct node {
	int p,v,id;
}a[N + 10];
int cmp(node x,node y) {
	return x.p < y.p;
}
int pre[N + 10],nxt[N + 10],ans[N + 10],tim[N + 10];
ll dd[N + 10];
int dis(int x,int y) {
	int npx = a[x].p,npy = a[y].p;
	return npx < npy ? npy - npx : m - npx + npy;
}
ll calc(int x,int y,int now) {
	ll ndx = dd[x] + 1ll * a[x].v * (now - tim[x]),ndy = dd[y] + 1ll * a[y].v * (now - tim[y]);
	if (a[x].id < a[y].id && dis(x,y) <= a[x].v) return 1;
	else if (a[x].id < a[y].id && a[x].v > a[y].v) return ceil((dis(x,y) - ndx + ndy - a[x].v) * 1.0 / (a[x].v - a[y].v)) + 1;
	return a[x].v <= a[y].v ? INF : ceil((dis(x,y) - ndx + ndy) * 1.0 / (a[x].v - a[y].v));
}
struct node2 {
	int x,y,v,w;
	bool operator < (const node2 &y) const {
		return v == y.v ? a[x].id > a[y.x].id : v > y.v;
	}
};
void del(int x) {
	pre[nxt[x]] = pre[x];
	nxt[pre[x]] = nxt[x];
	pre[x] = nxt[x] = -1;
}
signed main() {
	n = read();
	m = read();
	for (int i = 1;i <= n;i ++ ) a[i].p = read(), a[i].v = read(), a[i].id = i;
	sort(a + 1,a + n + 1,cmp);
	priority_queue<node2> q;
	for (int i = 1;i <= n;i ++ ) nxt[i] = i % n + 1, pre[i] = (i + n - 2) % n + 1, q.push({i,nxt[i],calc(i,nxt[i],0),vis[nxt[i]] = ++tot});
	while (q.size() > 1) {
		node2 t = q.top();
		if (pre[t.x] < 0 || t.y != nxt[t.x] || vis[t.y] != t.w) {
			q.pop();
			continue;
		}
		if (t.v >= INF) break;q.pop();
		int x = t.x,lsp = a[x].p;
		ll dx = 1ll * (t.v - tim[x]) * a[x].v;
		dd[x] += dx;
		tim[x] = t.v;
		a[x].v --;
		del(nxt[x]);
		while (nxt[x] != x && dd[x] - (dd[nxt[x]] + 1ll * (t.v - tim[nxt[x]] - (a[x].id < a[nxt[x]].id)) * a[nxt[x]].v) >= dis(x,nxt[x])) a[x].v --, del(nxt[x]);
		a[x].v = max(a[x].v,0ll);
		q.push({pre[x],x,t.v + calc(pre[x],x,t.v),vis[x] = ++tot});
		q.push({x,nxt[x],t.v + calc(x,nxt[x],t.v),vis[nxt[x]] = ++tot});
	}
	tot = 0;
	while (!q.empty()) {
		node2 t = q.top();
		if (!(pre[t.x] < 0 || t.y != nxt[t.x] || vis[t.y] != t.w)) ans[++tot] = a[t.x].id;
		q.pop();
	}
	sort(ans + 1,ans + tot + 1);
	printf("%lld\n",tot);
	for (int i = 1;i <= tot;i ++ ) printf("%lld ",ans[i]);
	return 0;
}
```
