---
title: CF E131 E. Text Editor
date: 2022-07-09 12:04:20
tags:
- dp
- 贪心
categories: ACM题解
mathjax: true
---

原题链接：[https://codeforces.com/contest/1701/problem/E](https://codeforces.com/contest/1701/problem/E)

## 题目大意

给你一个字符串 s，要你通过下面的操作变成字符串 t，同时要求操作数最小。（光标初始在 s 的最后一个字符的右边）

1. home 光标移动到第一个字符左边
2. left 和 right 光标移动一个单位距离
3. backspace 删除光标前的那个字符
4. end 光标移动到最后一个字符右边

## 思路分析

比赛时统计答案的部分没调出来。

一眼 dp。有一个很明显的结论：在确定了你要留下的东西之后，你先把右边该删的删完再去删左边要删的，显然是更优的。所以我们可以分开 dp，然后把从左往右的答案和从右往左的答案拼起来。

用 $f(i, j)$ 表示光标在i的右边，且从右往左匹配到第j个字母的最小操作数，$g(i,j)$ 表示光标在i的左边，且从左往右匹配到第j个字母的最小操作数。于是有下面的方程：

$$
f(i,j) = \begin{cases}
\inf, & s_i \neq t_j \\
n - i, & j = m \\
\min {f(k, j + 1) + k - i}, & s_k = t_{j + 1} \\
\end{cases}
$$
$$
g(i, j) = \begin{cases}
\inf, & s_i \neq t_j \\
1 + (i - 1) \times 2, & j = 1 \\
\min {g(k, j - 1) + (i - k) + (i - k - 1)}, & s_k = t_{j - 1} \\
\end{cases}
$$
从左到右扫求出 g，从右往左扫求出 f。直接枚举 k 的话是 $O(n^3)$。加一个数组来存下对于每个 j，$f(i, j) + i$ 和 $g(i, j) - i\times 2$ 的最小值，就可以优化到 $O(n^2)$。

然后统计答案。

统计答案的细节如下：

最直观的，我们需要取 $f(i, j) + g(i, j)$。

对于从右往左一路删到底的情况，我们在从右往左匹配到第一个字符后，还要考虑它前面是不是还有多余的字符要删，如果有，那我们需要 left 一次，然后删掉多余的字符。

即：$f(i, 1) + \min(i - 1, 1) + i - 1$。

然后，还有一种较为麻烦的情形：我们在 dp 时是默认光标跟着匹配过程移动的，可有的时候，中间那个连续段，我们并不需要把光标移进去，因为那段中间没有任何需要删的内容。我们删到那一段右边后就可以直接润去删它的左边了。

即：$f(i + k, j + k) + g(i, j)$，当 $s_{i,i+1,...,i+k} = t_{j,j+1,...,j+k}$ 时。

以及在这种情形下也有一个特判：$i = 1$ 且 $j = 1$，并不需要 $g(i, j)$ 里的那次 home。

即：$f(1 + k, 1 + k) + g(1, 1) - 1$

我们统计答案用的必须是有效的 $f$ 和 $g$，也就是说这两个数不可以是 $\inf$。

对于前面几种情形，直接 $O(n^2)$ 的枚举就可以统计。

对于中间有一段不需要把光标移进去的情形，我们可以想到一个很直观的三重循环：枚举 i, j 和 k。这样会 tle on test 20。所以我们需要加一点优化。

一个很明显的贪心结论：既然你中间要弄出一段来，那这一段一定是能弄多长就弄多长吧。

以及另一个明显的结论：如果 $f(i + k, j + k) = \inf$，根据我们的转移方程，$f(i + k + 1, j + k + 1)$ 显然不可能不是 $\inf$，不然它可以往左边的 i+k 转移使得前者不是 $\inf$。

由于我们是从左往右枚举 i 的。那如果从 i, j 这里凑出了一段来，那后面枚举到 i+1, j+1 的时候再去枚举 k 就没有必要了，因为不可能会比前面更优。

所以我们直接把扫过的用上的 $f(i+k, j+k)$ 设成 $\inf$。及时中断不需要的循环。

## 代码

不要吐槽为什么 rep 和 for 混着用。因为是比赛时敲的，顺手就敲进去了。

```c++
#include <cstdio>
#include <algorithm>
#include <cstring>
using namespace std;
typedef long long ll;
#define rep(i, a, b) for(int i = (a); i <= (b); i++)
const int N = 5e3 + 5, INF = 0x3f3f3f3f;
int n, m; char s[N], t[N]; int f[N][N], mi[N], g[N][N];
int main() {
    //freopen("in.txt", "r", stdin);
    int T; scanf("%d", &T);
    while(T--) {
        scanf("%d%d", &n, &m);
        for(int i = 1; i <= n; i++) for(int j = 1; j <= m; j++) g[i][j] = f[i][j] = INF;
        scanf(" %s %s", s + 1, t + 1);
        rep(i, 1, m) mi[i] = INF;
        for(int i = n; i >= 1; i--) {
            for(int j = m; j >= 1; j--) {
                if(s[i] != t[j]) continue;
                if(j == m) {
                    f[i][j] = n - i;
                    continue;
                }
                /*for(int k = i + 1; k <= n; k++) {
                    if(s[k] == t[j + 1]) f[i][j] = min(f[i][j], f[k][j + 1] + k - i);
                }*/
                if(mi[j + 1] != INF) f[i][j] = min(f[i][j], mi[j + 1] - i);
                else break;
            }
            for(int j = 1; j <= m; j++) mi[j] = min(mi[j], f[i][j] + i);
        }
        rep(i, 1, m) mi[i] = INF;
        for(int i = 1; i <= n; i++) {
            for(int j = 1; j <= m; j++) {
                if(s[i] != t[j]) continue;
                if(j == 1) {
                    g[i][j] = 1 + (i - 1) * 2;
                    continue;
                }
                /*for(int k = 1; k <= i - 1; k++) {
                    if(s[k] == t[j - 1]) g[i][j] = min(g[i][j], g[k][j - 1] + i - k + i - k - 1);
                }*/
                if(mi[j - 1] != INF) g[i][j] = min(g[i][j], mi[j - 1] + 2 * i);
                else break;
            }
            rep(j, 1, m) {
                if(g[i][j] == INF) continue;
                mi[j] = min(mi[j], g[i][j] - 2 * i - 1);
            }
        }
        int res = INF;
        for(int i = 1; i <= n; i++) {
            for(int j = 1; j <= m; j++) {
                if(s[i] != t[j]) continue;
                //printf("i=%d j=%d f=%d g=%d\n", i, j, f[i][j], g[i][j]);
                res = min(res, g[i][j] + f[i][j]);
                if(j == 1 && g[i][j] != INF) {
                    int d = i == 1 ? 0 : 1;
                    res = min(res, f[i][j] + d + i - 1); // 顺着删到后面去
                }
                /*if(j == m && f[i][j] != INF) {
                    res = min(res, g[i][j] + n - i); // 其实会被算过，不需要这个也行
                }*/
            }
        }
        //printf("ss%d\n", res);
        for(int i = 1; i <= n; i++) {
            for(int j = 1; j <= m; j++) {
                if(s[i] != t[j]) continue;
                for(int k = j + 1; k <= m; k++) {
                    if(s[i + k - j] != t[k]) break;
                    if(g[i][j] == INF || f[i + k - j][k] == INF) break;
                    //printf("i=%d %d j=%d %d: g=%d f=%d\n", i, i + k - j, j, k, g[i][j], f[i + k - 1][k]);
                    if(j == 1) {
                        int d = i == 1 ? 0 : 1;
                        res = min(res, f[i + k - j][k] + d + (i - 1) * 2);
                    }
                    /*if(k == m) {
                        res = min(res, g[i][j] + n - (i + k - j));
                    }*/
                    res = min(res, g[i][j] + f[i + k - j][k]);
                    f[i + k - j][k] = INF;
                }
            }
        }
        if(res == INF) printf("-1\n");
        else printf("%d\n", res);
    }
    return 0;
}
```