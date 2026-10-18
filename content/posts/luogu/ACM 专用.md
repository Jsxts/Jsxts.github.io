---
title: "ACM 专用"
date: 2024-03-30T10:08:46+08:00
draft: false
author: "Jsxts"
tags: ["个人记录"]
slug: "acm-专用"
original: "https://www.luogu.com.cn/article/7bmffs10"
---
看到一个加一个。

网络流：

```cpp
struct edge {
	int v,f,nxt;
}ed[N * 2000 + 10];
int head[N * 3 + 10],cnt = 1,S,T,h2[N * 3 + 10];
void add(int u,int v,int f) {
	ed[++cnt] = {v,f,head[u]};
	head[u] = cnt;
}
void add_(int u,int v,int f) {
	add(u,v,f), add(v,u,0);
}
int dep[N * 3 + 10];
bool bfs() {
	memset(dep,-1,sizeof dep);
	queue<int> q;
	q.push(S);
	dep[S] = 0;
	while (!q.empty()) {
		int u = q.front();q.pop();
		for (int i = h2[u] = head[u];i;i = ed[i].nxt) {
			int v = ed[i].v;
			if (!ed[i].f || dep[v] >= 0) continue;
			dep[v] = dep[u] + 1;
			q.push(v);
		}
	}
	return ~dep[T];
}
int dfs(int x,int fl) {
	if (x == T) return fl;
	int tmp = 0;
	for (int i = h2[x];i;i = h2[x] = ed[i].nxt) {
		int v = ed[i].v;
		if (!ed[i].f || dep[v] != dep[x] + 1) continue;
		int vfl = dfs(v,min(ed[i].f,fl - tmp));
		tmp += vfl;
		ed[i].f -= vfl, ed[i ^ 1].f += vfl;
		if (tmp == fl) break;
	}
	return tmp;
}
```


费用流：

```cpp
struct edge {
	int v,f,c,nxt;
}ed[N * 20 + 10];
int head[N + 10],cnt = 1,n,m,S,T,h2[N + 10];
void add(int u,int v,int f,int c) {
	ed[++cnt] = {v,f,c,head[u]};
	head[u] = cnt;
}
void add_(int u,int v,int f,int c) {
	add(u,v,f,c), add(v,u,0,-c);
}
ll dis[N + 10];
bool vis[N + 10];
bool spfa() {
	for (int i = 1;i <= n;i ++ ) dis[i] = INF, vis[i] = 0;
	dis[S] = 0;
	queue<int> q;
	q.push(S);
	while (!q.empty()) {
		int u = q.front();q.pop();
		vis[u] = 0;
		for (int i = h2[u] = head[u];i;i = ed[i].nxt) {
			int v = ed[i].v;
			if (!ed[i].f || dis[v] <= dis[u] + ed[i].c) continue;
			dis[v] = dis[u] + ed[i].c;
			if (!vis[v]) vis[v] = 1, q.push(v);
		}
	}
	return dis[T] != INF;
}
ll ans;
int dfs(int x,int fl) {
	if (x == T) return fl;
	vis[x] = 1;
	int tmp = 0;
	for (int i = h2[x];i;i = h2[x] = ed[i].nxt) {
		int v = ed[i].v;
		if (!ed[i].f || dis[v] != dis[x] + ed[i].c || vis[v]) continue;
		int vfl = dfs(v,min(ed[i].f,fl - tmp));
		tmp += vfl;
		ans += 1ll * ed[i].c * vfl;
		ed[i].f -= vfl, ed[i ^ 1].f += vfl;
		if (tmp == fl) break;
	}
	vis[x] = 0;
	return tmp;
}
```


平面向量：

```cpp
const double eps = 1e-6;
int D(double x) {
	return fabs(x) <= eps ? 0 : x > 0 ? 1 : -1;
}
struct vec {
	double x,y;
	vec(double _x = 0,double _y = 0) {
		x = _x, y = _y;
	}
	bool operator == (const vec &a) {return !D(x - a.x) && !D(y - a.y);}
	vec operator + (const vec &a) {return vec(x + a.x,y + a.y);}
	void operator += (const vec &a) {x += a.x, y += a.y;}
	vec operator - (const vec &a) {return vec(x - a.x,y - a.y);}
	double operator * (const vec &a) {return x * a.x + y * a.y;}
	vec operator * (const double &k) {return vec(x * k,y * k);}
	vec operator / (const double &k) {return vec(x / k,y / k);}
	double len() {
		return sqrt(x * x + y * y);
	}
	vec unit() {
		return *this / len();
	}
}a[N + 10];
double dis(vec x,vec y) {
	return sqrt((double)(x.x - y.x) * (x.x - y.x) + (double)(x.y - y.y) * (x.y - y.y));
}
//叉积正负：cross(a,b)>0:a逆时针到b的夹角<180 
double cross(vec x,vec y) {
	return x.x * y.y - x.y * y.x;
}
struct Line {
	vec x,v;
};
Line get(vec x,vec y) {
	return {x,y - x};
}
vec inter(Line x,Line y) {//记得判共线！ 
	double t = fabs(cross((y.x - x.x),y.v)) / fabs(cross(x.v,y.v));
	return x.x + x.v * t;
}
vec getpro(vec x,vec y) {//投影 
	return x.unit() * (x * y / x.len());
}
double dis(Line x,vec y) {
	vec c = getpro(x.v,y - x.x);
	return (y - x.x - c).len();
}
vector<vec> Get_Convex(vector<vec> &a) {
	sort(a.begin(),a.end(),[](const vec &x,const vec &y){return !D(x.x - y.x) ? x.y < y.y : x.x < y.x;});
	vector<vec> b;
	int n = a.size();
	for (int i = 0;i < n;i ++ ) {
		while (b.size() > 1 && D(cross(b.back() - b[b.size() - 2],a[i] - b[b.size() - 2])) <= 0) b.pop_back();
		b.push_back(a[i]);
	}
	int t = b.size();
	for (int i = n - 2;~i;i -- ) {
		while (b.size() > t && D(cross(b.back() - b[b.size() - 2],a[i] - b[b.size() - 2])) <= 0) b.pop_back();
		b.push_back(a[i]);
	}
	b.pop_back();
	return b;
}
vector<vec> Minkowski(vector<vec> a,vector<vec> b) {//a,b为凸包 
	vector<vec> c,d,res;
	for (int i = 0;i < a.size() - 1;i ++ ) c.push_back(a[i + 1] - a[i]);c.push_back(a[0] - a.back());
	for (int i = 0;i < b.size() - 1;i ++ ) d.push_back(b[i + 1] - b[i]);d.push_back(b[0] - b.back());
	int t1 = 0,t2 = 0,n = a.size(),m = b.size();
	vec now = a[0] + b[0];
	for (int i = 0;i < n + m;i ++ ) {
		if (t2 >= m || (t1 < n && D(cross(c[t1],d[t2])) >= 0)) now += c[t1++], res.push_back(now);
		else now += d[t2 ++], res.push_back(now);
	}
	res = Get_Convex(res);
	return res;
}
vector<vec> c;

bool in(vector<vec> &a,vec x) {//点是否在凸包内，须先将坐标系平移，使得左下角位于原点 
	if (D(cross(x,a[1])) > 0 || D(cross(a.back(),x)) > 0) return 0;
	int pos = lower_bound(a.begin(),a.end(),x,[](vec x,vec y) {return D(cross(x,y)) > 0 || (!D(cross(x,y)) && x.len() < y.len());}) - a.begin() - 1;
	return D(cross(x - a[pos],a[(pos + 1) % a.size()] - a[pos])) <= 0;
}
```

ST 表 LCA：
```cpp
int n,q,cnt,head[N + 10];
struct edge {
	int v,nxt;
}ed[N * 2 + 10];
void add(int u,int v) {
	ed[++cnt] = {v,head[u]};
	head[u] = cnt;
}
pair<int,int> st[20][N + 10];
int tot,dfn[N + 10],dep[N + 10];
void dfs(int x,int ff) {
	st[0][tot] = {dep[x],ff}, dfn[x] = ++tot;
	for (int i = head[x];i;i = ed[i].nxt) {
		int v = ed[i].v;
		if (v == ff) continue;
		dep[v] = dep[x] + 1;
		dfs(v,x);
	}
}
int lg[N + 10];
int LCA(int x,int y) {
	if (x == y) return x;
	x = dfn[x], y = dfn[y];
	if (x > y) swap(x,y);y --;
	int len = lg[y - x + 1];
	return min(st[len][x],st[len][y-(1<<len)+1]).y;
}
int main() {
  
	for (int i = 2;i <= n;i ++ ) lg[i] = lg[i >> 1] + 1;
	dfs(1,0);
	for (int i = 1;i <= 17;i ++ )
		for (int j = 1;j + (1 << i) - 1 <= n;j ++ )
			st[i][j] = min(st[i - 1][j],st[i - 1][j + (1 << i - 1)]);
}
```

区间推平区间求和线段树：

```cpp

struct Seg {
	int l,r,v,tag;
}t[N * 4 + 10];
void build(int x,int l,int r) {
	t[x].l = l, t[x].r = r, t[x].tag = inf;
	if (l == r) return;
	int mid = l + r >> 1;
	build(x<<1,l,mid), build(x<<1|1,mid + 1,r);
}
void pushup(int x) {
	t[x].v = t[x<<1].v + t[x<<1|1].v;
}
void f(int x,int s) {
	t[x].tag = s;
	t[x].v = (t[x].r - t[x].l + 1) * s;
}
void pushdown(int x) {
	if (t[x].tag != inf) f(x<<1,t[x].tag), f(x<<1|1,t[x].tag), t[x].tag = inf;
}
void modify(int x,int l,int r,int s) {
	if (t[x].l >= l && r >= t[x].r) {
		f(x,s);
		return;
	}
	pushdown(x);
	int mid = t[x].l + t[x].r >> 1;
	if (mid >= l) modify(x<<1,l,r,s);
	if (mid < r) modify(x<<1|1,l,r,s);
	pushup(x);
}
int query(int x,int l,int r) {
	if (t[x].l >= l && r >= t[x].r) return t[x].v;
	int mid = t[x].l + t[x].r >> 1;
	pushdown(x);
	int s=0;
	if (mid >= l) s=query(x<<1,l,r);
	if (mid < r) s+=query(x<<1|1,l,r);
	return s;
}
```

矩阵快速幂：
```cpp
struct Matrix {
	int w[3][3];
	Matrix() {
		memset(w,0,sizeof w);
	}
	int* operator[] (const int &x) {
		return w[x];
	}
	friend Matrix operator * (Matrix x,Matrix y) {
		Matrix z;
		for (int i = 0;i < 3;i ++ )
			for (int k = 0;k < 3;k ++ )
				for (int j = 0;j < 3;j ++ )
					z[i][j] = (z[i][j] + 1ll * x[i][k] * y[k][j]) % mod;
		return z;
	}
}A;
Matrix mpow(Matrix a,int b) {
	Matrix res;
	for (int i = 0;i < 3;i ++ ) res[i][i] = 1;
	while (b) {
		if (b & 1) res = res * a;
		b >>= 1, a = a * a;
	}
	return res;
}
```

Miller-Rabin:
```cpp
ll tab[] = {2, 325, 9375, 28178, 450775, 9780504, 1795265022};
bool MR(ll n) {
	if (n <= 1) return 0;
	ll x = n - 1,t = 0;
	while (x % 2 == 0) x /= 2, t ++;
	for (int i = 0;i < 7;i ++ ) {
		ll a = tab[i];
		ll p = qpow(a,x,n);
		if (p <= 1) continue;
		for (int j = 1;j <= t;j ++ ) {
			if (p == n - 1) {
				goto bre;
			}
			p = mul(p,p,n);
		}
		return 0;
		bre:;
	}
	return 1;
}
```

轻重链剖分：

```cpp
int n,q,head[N + 10],cnt,sz[N + 10],son[N + 10],dep[N + 10],top[N + 10],fa[N + 10],id[N + 10],rnk[N + 10],tot,End[N + 10];
struct edge {
	int v,nxt;
}ed[N * 2 + 10];
void add(int u,int v) {
	ed[++cnt] = {v,head[u]};
	head[u] = cnt;
}
void dfs(int x,int f,int d) {
	fa[x] = f, dep[x] = d, sz[x] = 1;
	for (int i = head[x];i;i = ed[i].nxt) {
		int v = ed[i].v;
		if (v == f) continue;
		dfs(v,x,d + 1);
		sz[x] += sz[v];
		if (sz[son[x]] < sz[v]) son[x] = v;
	}
}
void dfs2(int x,int tp) {
	id[x] = ++tot, top[x] = tp, rnk[tot] = x;
	End[tp] = x;
	if (!son[x]) return;
	dfs2(son[x],tp);
	for (int i = head[x];i;i = ed[i].nxt) {
		int v = ed[i].v;
		if (v == fa[x] || v == son[x]) continue;
		dfs2(v,v);
	}
}
```

莫队：

```cpp
const int B = 300;
int n,m,a[N + 10];
struct ask {
	int l,r,id;
	bool operator < (const ask &x) const {
		return x.l / B == l / B ? r < x.r : l < x.l;
	}
}q[N + 10];
```

船新 NTT 全家桶：

```cpp
typedef vector<int> poly;
namespace Poly {
	const int G = 3,invG = qpow(G,mod - 2);
	poly ro,w;
	int tn;
	void init(int n) {
		if (n == tn) return;
		tn = n;
		if (ro.size() < n) ro.resize(n);
		for (int i = 0;i < n;i ++ ) {
			ro[i] = (ro[i >> 1] >> 1) | (i & 1 ? n / 2 : 0);
		}
	}
	typedef unsigned long long ull;
	void NTT(poly &f,bool fl,int n) {
		vector<ull> g;
		g.resize(n);
		for (int i = 0;i < n;i ++ ) g[i] = f[ro[i]];
		for (int len = 1;len < n;len <<= 1) {
			int w0 = qpow(fl ? G : invG,(mod - 1) / 2 / len);w[0] = 1;
			for (int j = 1;j < len;j ++ ) w[j] = 1ll * w[j - 1] * w0 % mod;
			for (int j = 0;j < n;j += 2 * len)
				for (int k = j;k < j + len;k ++ ) {
					ull g0 = g[k],g1 = g[k + len];
					g[k] = (g0 + 1ll * w[k - j] * g1 % mod);
					g[k + len] = (g0 - 1ll * w[k - j] * g1 % mod + mod);
				}
			if (len == 1 << 17)
				for (int j = 0;j < n;j ++ ) g[j] %= mod;
		}
		if (!fl) {
			int inv = qpow(n,mod - 2);
			for (int i = 0;i < n;i ++ ) f[i] = g[i] % mod * inv % mod;
		}
		else for (int i = 0;i < n;i ++ ) f[i] = g[i] % mod;
	}
	poly operator * (poly x,poly y) {
		int n = x.size(),m = y.size(),lim;
		for (lim = 1;lim < n + m - 1;lim <<= 1);
		init(lim);
		x.resize(lim), y.resize(lim); w.resize(lim);
		NTT(x,1,lim), NTT(y,1,lim);
		for (int i = 0;i < lim;i ++ ) x[i] = 1ll * x[i] * y[i] % mod;
		NTT(x,0,lim);
		x.resize(n + m - 1);
		return x;
	}
	poly operator + (const poly &x,const poly &y) {
		poly t = x;
		t.resize(max(x.size(),y.size()));
		for (int i = 0;i < y.size();i ++ ) t[i] = getmod(t[i] + y[i]);
		return t;
	}
	poly operator - (const poly &x,const poly &y) {
		poly t = x;
		t.resize(max(x.size(),y.size()));
		for (int i = 0;i < y.size();i ++ ) t[i] = getmod(t[i] - y[i] + mod);
		return t;
	}
	poly operator * (poly x,const int &y) {
		for (int &i : x) i = 1ll * i * y % mod;
		return x;
	}
	poly inv(poly x) {
		poly y = {qpow(x[0],mod - 2)};
		int n = x.size(),lim;
		for (int i = 1;y.size() < n;i ++ ) {
			poly tx(x.begin(),x.begin() + min(n,(1 << i)));
			lim = 1 << i + 1;
			init(lim);
			y.resize(lim), tx.resize(lim); w.resize(lim);
			NTT(y,1,lim), NTT(tx,1,lim);
			for (int j = 0;j < lim;j ++ ) y[j] = 1ll * y[j] * (2 - 1ll * y[j] * tx[j] % mod + mod) % mod;
			NTT(y,0,lim);
			y.resize(1 << i);
		}
		y.resize(n);
		return y;
	}
	poly operator / (poly x,poly y) {
		int n = x.size(),m = y.size();
		reverse(x.begin(),x.end()), reverse(y.begin(),y.end());
		x.resize(n - m + 1), y.resize(n - m + 1);
		poly tq = x * inv(y);tq.resize(n - m + 1);
		reverse(tq.begin(),tq.end());
		return tq;
	}
	poly operator % (poly x,poly y) {
		return x - (x / y) * y;
	}
	poly sqrt(poly x) {
		poly y = {1};
		int n = x.size();
		for (int i = 1;y.size() < n * 2;i ++ ) {
			y = (x + y * y) * inv(y * 2);
			y.resize(1 << i);
		}
		y.resize(n);
		return y;
	}
	poly der(poly x) {
		for (int i = 0;i < x.size() - 1;i ++ ) x[i] = (i + 1ll) * x[i + 1] % mod;
		x.pop_back();
		return x;
	}
	poly integ(poly x) {
		x.push_back(0);
		int n = x.size();
		poly inv;inv.resize(n + 2);
		inv[0] = inv[1] = 1;
		for (int i = 2;i <= n + 1;i ++ ) inv[i] = 1ll * (mod - mod / i) * inv[mod % i] % mod;
		for (int i = n - 1;i;i -- ) x[i] = 1ll * inv[i] * x[i - 1] % mod;
		x[0] = 0;
		return x;
	}
	poly ln(const poly &x) {
//		assert(x[0] == 1);
		poly y = integ(der(x) * inv(x));
		return {y.begin(),y.begin() + x.size()};
	}
	poly exp(poly x) {
		poly y = {1};
		int n = x.size();
		for (int i = 1;(1 << i - 1) < n * 2;i ++ ) {
			y = y - (y * (ln(y) - x));
			y.resize(1 << i);
		}
		y.resize(n);
		return y;
	}
	poly ppow(poly x,int k,int k2) {
		int t = x[0];
		return exp(ln(x * qpow(t,mod - 2)) * k) * qpow(t,k2);
	}
}using namespace Poly;
```

任模全家桶：

```
typedef vector<int> poly;
namespace Poly {
	struct modNTT {
		int mod,G,invG;
		void init(int _mod,int _G) {
			mod = _mod, G = _G;
			invG = qpow(G,mod - 2,mod);
		}
		poly ro,w;
		int tn;
		void init(int n) {
			if (n == tn) return;
			tn = n;
			if (ro.size() < n) ro.resize(n);
			for (int i = 0;i < n;i ++ ) {
				ro[i] = (ro[i >> 1] >> 1) | (i & 1 ? n / 2 : 0);
			}
		}
		typedef unsigned long long ull;
		void NTT(poly &f,bool fl,int n) {
			vector<ull> g;
			g.resize(n);
			for (int i = 0;i < n;i ++ ) g[i] = f[ro[i]];
			for (int len = 1;len < n;len <<= 1) {
				int w0 = qpow(fl ? G : invG,(mod - 1) / 2 / len,mod);w[0] = 1;
				for (int j = 1;j < len;j ++ ) w[j] = 1ll * w[j - 1] * w0 % mod;
				for (int j = 0;j < n;j += 2 * len)
					for (int k = j;k < j + len;k ++ ) {
						ull g0 = g[k],g1 = g[k + len];
						g[k] = (g0 + 1ll * w[k - j] * g1 % mod);
						g[k + len] = (g0 - 1ll * w[k - j] * g1 % mod + mod);
					}
				if (len == 1 << 17)
					for (int j = 0;j < n;j ++ ) g[j] %= mod;
			}
			if (!fl) {
				int inv = qpow(n,mod - 2,mod);
				for (int i = 0;i < n;i ++ ) f[i] = g[i] % mod * inv % mod;
			}
			else for (int i = 0;i < n;i ++ ) f[i] = g[i] % mod;
		}
		poly mul(poly x,poly y) {
			int n = x.size(),m = y.size(),lim;
			for (lim = 1;lim < n + m - 1;lim <<= 1);
			init(lim);
			x.resize(lim), y.resize(lim); w.resize(lim);
			NTT(x,1,lim), NTT(y,1,lim);
			for (int i = 0;i < lim;i ++ ) x[i] = 1ll * x[i] * y[i] % mod;
			NTT(x,0,lim);
			x.resize(n + m - 1);
			return x;
		}
	}m[3];
	poly operator * (poly x,poly y) {
		poly res[3];
		int n = x.size(),mm = y.size();
		for (int i = 0;i < 3;i ++ ) res[i] = m[i].mul(x,y);
		poly z;z.resize(n + mm - 1);
		int inv1 = qpow(m[0].mod,m[1].mod - 2,m[1].mod);
		int inv2 = qpow(1ll * m[0].mod * m[1].mod % m[2].mod,m[2].mod - 2,m[2].mod);
		for (int i = 0;i < n + mm - 1;i ++ ) {
			ll k1 = 1ll * (res[1][i] - res[0][i] + m[1].mod) * inv1 % m[1].mod;
			ll x = res[0][i] + 1ll * k1 * m[0].mod;
			ll k4 = (res[2][i] - x % m[2].mod + m[2].mod) % m[2].mod * inv2 % m[2].mod;
			z[i] = (x + k4 % mod * m[0].mod % mod * m[1].mod) % mod;
		}
		return z;
	}
	poly operator + (const poly &x,const poly &y) {
		poly t = x;
		t.resize(max(x.size(),y.size()));
		for (int i = 0;i < y.size();i ++ ) t[i] = getmod(t[i] + y[i]);
		return t;
	}
	poly operator - (const poly &x,const poly &y) {
		poly t = x;
		t.resize(max(x.size(),y.size()));
		for (int i = 0;i < y.size();i ++ ) t[i] = getmod(t[i] - y[i] + mod);
		return t;
	}
	poly operator * (poly x,const int &y) {
		for (int &i : x) i = 1ll * i * y % mod;
		return x;
	}
	poly inv(poly x) {
		poly y = {qpow(x[0],mod - 2)};
		int n = x.size();
		for (int i = 1;y.size() < n;i ++ ) {
			y = y * 2 - y * y * x;
			y.resize(1 << i);
		}
		y.resize(n);
		return y;
	}
	poly operator / (poly x,poly y) {
		int n = x.size(),m = y.size();
		reverse(x.begin(),x.end()), reverse(y.begin(),y.end());
		x.resize(n - m + 1), y.resize(n - m + 1);
		poly tq = x * inv(y);tq.resize(n - m + 1);
		reverse(tq.begin(),tq.end());
		return tq;
	}
	poly operator % (poly x,poly y) {
		return x - (x / y) * y;
	}
	poly sqrt(const poly &x) {
		poly y = {1};
		int n = x.size();
		for (int i = 1;y.size() < n * 2;i ++ ) {
			y = (x + y * y) * inv(y * 2);
			y.resize(1 << i);
		}
		y.resize(n);
		return y;
	}
	poly der(poly x) {
		for (int i = 0;i < x.size() - 1;i ++ ) x[i] = (i + 1ll) * x[i + 1] % mod;
		x.pop_back();
		return x;
	}
	poly integ(poly x) {
		x.push_back(0);
		int n = x.size();
		poly inv;inv.resize(n + 2);
		inv[0] = inv[1] = 1;
		for (int i = 2;i <= n + 1;i ++ ) inv[i] = 1ll * (mod - mod / i) * inv[mod % i] % mod;
		for (int i = n - 1;i;i -- ) x[i] = 1ll * inv[i] * x[i - 1] % mod;
		x[0] = 0;
		return x;
	}
	poly ln(const poly &x) {
//		assert(x[0] == 1);
		poly y = integ(der(x) * inv(x));
		return {y.begin(),y.begin() + x.size()};
	}
	poly exp(poly x) {
		poly y = {1};
		int n = x.size();
		for (int i = 1;(1 << i - 1) < n * 2;i ++ ) {
			y = y - (y * (ln(y) - x));
			y.resize(1 << i);
		}
		y.resize(n);
		return y;
	}
	poly ppow(poly x,int k,int k2) {
		int t = x[0];
		return exp(ln(x * qpow(t,mod - 2)) * k) * qpow(t,k2);
	}
}using namespace Poly;
int main() {
	m[0].init(998244353,3);
	m[1].init(1004535809,3);
	m[2].init(469762049,3);
}
```
