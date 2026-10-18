---
title: "CF335B Palindrome"
date: 2023-11-16T10:11:12+08:00
draft: false
author: "Jsxts"
tags: ["题解"]
slug: "cf335b-palindrome"
original: "https://www.luogu.com.cn/article/2xq49cad"
math: true
---
另类 DP。

设 $dp_{i,j}$ 表示 前 $i$ 个字符，组成的回文子序列长度为 $j$ 时，子序列的起点的最大值为多少。显然起点越大越优，所以这个 DP 是正确的。

转移很简单，分类讨论选或不选 $i$：

- 不选，$dp_{i,j}=dp_{i-1,j}$。

- 选，$dp_{i,j}=pre_{dp_{i-1,j-2}-1,s_i}$，其中 $pre_{i,j}$ 表示 $i$ 前第一个字符 $j$ 的位置，表示在之前整个子序列的前后同时选了个 $s_i$。

初始化即 $dp_{i,1}=i,dp_{i,2}=\max(dp_{i-1,2},pre_{i-1,s_i})$。

由于还要记录方案，在 DP 时多记录一个数表示当前回文串的结尾，然后倒着推即可。

不需要特判 $s$ 的长度。

代码：

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;
const int inf = 1e9;
const ll INF = 1e15;
const int N = 5e4;
inline int read() {
	int s = 0,f = 1;char ch = getchar();
	while (!isdigit(ch)) f = ch == '-' ? -1 : 1, ch = getchar();
	while (isdigit(ch)) s = (s << 3) + (s << 1) + ch - '0', ch = getchar();
	return s*f;
}
char s[N + 10];
pair<int,int> dp[N + 10][110];
int n,pre[N + 10][26];
int main() {
	scanf("%s",s + 1);
	n = strlen(s + 1);
	for (int i = 1;i <= n;i ++ ) {
		for (int j = 0;j < 26;j ++ ) pre[i][j] = pre[i - 1][j];
		pre[i][s[i] - 'a'] = i;
	}
	for (int i = 1;i <= n;i ++ ) {
		dp[i][1] = {i,i}, dp[i][2] = max(dp[i - 1][2],{pre[i - 1][s[i] - 'a'],i});
		for (int j = 3;j <= min(i,100);j ++ ) {
			dp[i][j] = dp[i - 1][j];
			if (dp[i - 1][j - 2].first) dp[i][j] = max(dp[i][j],{pre[dp[i - 1][j - 2].first - 1][s[i] - 'a'],i});
		}
//		cout << "dp:" << i << endl;
//		for (int j = 0;j <= i;j ++ ) cout << j << ' ' << dp[i][j].first << ' ' << dp[i][j].second << endl;
	}
	for (int i = 100;i;i -- ) {
		if (dp[n][i].first > 0) {
//			cout << i << endl;
			int t = dp[n][i].second,j = i;
			string ans = "";
			while (j > 0) {
				ans += s[t];
				if (j > 1) j -= 2, t = dp[t - 1][j].second;
				else break;
			}
			cout << ans;
			for (int i = ans.size() - 1 - (j % 2);~i;i -- ) putchar(ans[i]);
			return 0;
		}
	}
	return 0;
}
```
