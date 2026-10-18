---
title: "KTT"
date: 2024-05-01T15:02:37+08:00
draft: false
author: "Jsxts"
tags: ["个人记录"]
slug: "ktt"
original: "https://www.luogu.com.cn/article/z5q0feuu"
math: true
---
## KTT

**KTT：维护 $N$ 个一次函数在各自的 $x=a_i$ 时的某些信息，支持将 $a_i$ 区间加正数，区间查询。**

KTT 维护区间最大子段和、区间加正数：EI的第六分块。

考虑最大子段和需要维护什么？$ls,rs,s,sum$ 表示前缀和最大值，后缀和最大值，最大子段和，区间和。

现在我们将这些值都看成一次函数 $y=kx+b$，其中对于一个值，它的 $k$ 就是它选择的长度。现在我们使得每个一次函数在 $x=0$ 时的值为它原本维护的值（前缀和最大值、后缀和最大值等），所以 $b=y-kx=y$，即 $b$ 为原本维护的值。

用线段树维护这些东西。考虑如何 pushup ？

一般的最大子段和的 pushup：

- $c.ls = \max(a.ls,a.sum + b.ls)$
- $c.rs = \max(b.rs,b.sum + a.rs)$
- $c.s = \max(a.s,b.s,a.rs + b.ls)$
- $c.sum = a.sum + b.sum$

现在放到一次函数上，共有取 $\max$ 和加两种不同运算。加法就是一次函数相加，取 $\max$ 就是比较一次函数在 $x=0$ 时的大小。

现在考虑区间加 $s$。对于一个完整被修改的区间，若它最大子段和选择的区间没有改变，则每个值会分别加上 $ks$，在一次函数中也就是 $x$ 坐标加 $s$。发现这些一次函数取 $\max$ 时会有交点，也就是说 $x$ 过了一些交点后维护的值大小关系会改变，进而维护的值发生改变。我们考虑判断什么时候 $x$ 坐标会超过交点，如果超过交点就重新计算整颗子树。

于是我们在 pushup 时还要维护所有取 $\max$ 时两个一次函数的交点横坐标的最小值 $mn$。当区间加时，将 $x$ 坐标加 $s$ 等同于将 $mn$ 减 $s$ 再把 $b$ 加上 $ks$（这样维护的值还是对的）。于是当 $mn<0$ 时从这个点开始往下找所有 $mn-s<0$ 的节点将其重构。

复杂度分析：[Ntokisq的博客](https://www.luogu.com.cn/blog/203623/guan-yu-ktt)

维护 max 的板子：CF1830F The Third Grace

维护 max 同理，只需要维护出子树内所有一次函数的交点中在 $>0$ 部分交点的最小横坐标，若 $x$ 坐标超过这个数了就要进入子树重构。

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;
const int inf = 1e9;
const ll INF = 1e18;
const int N = 1e6;
inline int read() {
	int s = 0,f = 1;char ch = getchar();
	while (!isdigit(ch)) f = ch == '-' ? -1 : 1, ch = getchar();
	while (isdigit(ch)) s = (s << 3) + (s << 1) + ch - '0', ch = getchar();
	return s*f;
}
int n,m,a[N + 10];
struct Line {
	ll k,b;
	inline friend Line operator + (Line a,Line b) {
		return {a.k + b.k,a.b + b.b};
	}
	inline friend bool operator < (Line a,Line b) {
		return a.b < b.b;
	}
	inline void add(ll y) {
		b += k * y;
	}
};
ll cross(Line x,Line y) {
	return (x.k == y.k || (x.b - y.b) / (y.k - x.k) < 0) ? INF : (x.b - y.b) / (y.k - x.k);
}
struct node {
	Line mx;
	ll x;
	inline friend node operator + (node a,node b) {
		node c;
		c.x = min(a.x,b.x);
		c.mx = max(a.mx,b.mx);
		c.x = min(c.x,cross(a.mx,b.mx));
		return c;
	}
};
struct Seg {
	int l,r;
	node v;
	ll tag;
}t[N * 4 + 10];
void pushup(int x) {
	t[x].v = t[x<<1].v + t[x<<1|1].v;
}
void build(int x,int l,int r) {
	t[x].l = l, t[x].r = r, t[x].tag = 0;
	if (l == r) {
		t[x].v = {{a[l],0},INF};
		return;
	}
	int mid = l + r >> 1;
	build(x<<1,l,mid), build(x<<1|1,mid + 1,r);
	pushup(x);
}
void f(int x,ll s) {
	t[x].tag += s;
	t[x].v.x -= s;
	t[x].v.mx.add(s);
}
void pushdown(int x) {
	if (t[x].tag) f(x<<1,t[x].tag), f(x<<1|1,t[x].tag);
	t[x].tag = 0;
}
void rebuild(int x,ll s) {
	if (t[x].v.x - s >= 0) {
		f(x,s);
		return;
	}
	rebuild(x<<1,s + t[x].tag), rebuild(x<<1|1,s + t[x].tag);
	t[x].tag = 0;
	pushup(x);
}
void modify(int x,int l,int r,int s) {
	if (t[x].l >= l && r >= t[x].r) {
		rebuild(x,s);
		return;
	}
	pushdown(x);
	int mid = t[x].l + t[x].r >> 1;
	if (mid >= l) modify(x<<1,l,r,s);
	if (mid < r) modify(x<<1|1,l,r,s);
	pushup(x);
}
void modify2(int x,int k,ll s) {
	if (t[x].l == t[x].r) {
		t[x].v.mx.b += s;
		return;
	}
	pushdown(x);
	int mid = t[x].l + t[x].r >> 1;
	if (mid >= k) modify2(x<<1,k,s);
	else modify2(x<<1|1,k,s);
	pushup(x);
}
node query(int x,int l,int r) {
	if (t[x].l >= l && r >= t[x].r) return t[x].v;
	pushdown(x);
	int mid = t[x].l + t[x].r >> 1;
	if (mid < l) return query(x<<1|1,l,r);
	if (mid >= r) return query(x<<1,l,r);
	return query(x<<1,l,r) + query(x<<1|1,l,r);
}
vector<int> vc[N + 10];
ll dp[N + 10];
int main() {
	int T = read();
	while (T -- ) {
		n = read(), m = read();
		for (int i = 1;i <= m + 1;i ++ ) vc[i].clear();
		for (int i = 1,l,r;i <= n;i ++ ) l = read(), r = read(), vc[r].push_back(l);
		for (int i = 1;i <= m;i ++ ) a[i] = read();
		build(1,0,m);
		for (int i = 1;i <= m + 1;i ++ ) {
			if (i > 1) dp[i] = query(1,0,i - 1).mx.b;
			modify2(1,i,dp[i]);
			for (auto j : vc[i]) modify(1,j,i,1);
		}
		printf("%lld\n",dp[m + 1]);
	}
	return 0;
}
```


一个只有全局加正数的特殊情况：此时可以在全局设一个 $now$ 而不是节点里面，比较两条一次函数的大小时改为比较 $kx+b$。可以少一个 $\log$（？

```cpp
struct Line {
	ll k,b;
	inline friend Line operator + (Line a,Line b) {
		return {a.k + b.k,a.b + b.b};
	}
	inline friend bool operator < (Line a,Line b) {
		return a.b + now * a.k < b.b + now * b.k;
	}
	inline void add(ll y) {
		b += k * y;
	}
};
inline ll cross(Line x,Line y) {
	return (x.k <= y.k) ? INF : (x.b - y.b + y.k - x.k - 1) / (y.k - x.k);
}
struct node {
	Line mx;
	ll x;
	inline friend node operator + (const node &a,const node &b) {
		node c;
		Line ta = a.mx,tb = b.mx;
		c.x = min(a.x,b.x);
		if (ta < tb) swap(ta,tb);
		c.mx = ta;
		
		c.x = min(c.x,cross(tb,ta));
		return c;
	}
};
struct Seg {
  int l,r;
  node v;
}t[N * 4 + 10];
inline void pushup(int x) {
  t[x].v = t[x<<1].v + t[x<<1|1].v;
}
void build(int x,int l,int r) {
  t[x].l = l, t[x].r = r;
  if (l == r) {
    t[x].v = {{-inf,-INF},INF};
    return;
  }
  int mid = l + r >> 1;
  build(x<<1,l,mid), build(x<<1|1,mid + 1,r);
  pushup(x);
}
inline void f(int x,ll s) {
  t[x].v.x -= s;
  t[x].v.mx.add(s);
}
void rebuild(int x) {
  if (t[x].v.x - now > 0 || t[x].l == t[x].r) {
    return;
  }
  rebuild(x<<1), rebuild(x<<1|1);
  pushup(x);
}
void modify2(int x,int k,int f) {
  if (t[x].l == t[x].r) {
    t[x].v = f ? (node){val[k],INF} : (node){{-inf,-INF},INF};
    return;
  }
  int mid = t[x].l + t[x].r >> 1;
  if (mid >= k) modify2(x<<1,k,f);
  else modify2(x<<1|1,k,f);
  pushup(x);
}
ll query(int x,int l,int r) {
  if (l > r) return -INF;
  if (t[x].l >= l && r >= t[x].r) return t[x].v.mx.k * now + t[x].v.mx.b;
  int mid = t[x].l + t[x].r >> 1;
  if (mid < l) return query(x<<1|1,l,r);
  if (mid >= r) return query(x<<1,l,r);
  return max(query(x<<1,l,r) , query(x<<1|1,l,r));
}
```
