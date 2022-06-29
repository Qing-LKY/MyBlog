---
title: CF#803 E. PermutationForces II
date: 2022-06-29 22:09:26
tags:
- 构造
- 排列计数
categories: ACM题解
mathjax: true
---

原题链接：[https://codeforces.com/contest/1700/problem/E](https://codeforces.com/contest/1700/problem/E)

## 题目大意

给你一个排列 A 和排列 B，你要通过下面的操作把 A 弄成 B。你有一个“力量”值 $s$，对于**每个** $1 \leq i \leq n$，选择**一个** $i \leq j \leq i + s$，交换 $i$ 和 $j$ 的**位置**。例如 1 3 2 交换 1 和 2，结果是 2 3 1。问你 A 能变成的不同 B 有多少种（给出了 B 的部分位置的值，其它位置的值可以随便取，只要是个排列就行）。

## 思路分析

这题太抽象了。比赛时连怎么换的都绕不清楚，一直满脑子的什么连有向边什么环什么链什么玩意，更别提计数了。

首先，因为从头到尾换的都是值，所以我们可以把其中一个排列“标准化”。让我们来想一想，如果手头有确定的排列 A 和排列 B，我要怎么把 A 换到 B 呢？

例如：A: 5 4 2 3 1, B: 3 1 4 2 5

将 B 标准化后：A: 4 3 5 2 1, B: 1 2 3 4 5

此时我们从左往右看，首先要把 1 弄到正确的位置上，于是我们交换且只能交换 1 和 4（因为每个值只能进行一个操作，且这个操作的对象必须是比它大的数，如果你现在不换 1，后面 1 就不会再被换到了）。此时变成了：A: 1 3 5 2 4, B: 1 2 3 4 5

换完 1 之后，1 显然是不会再被动的（因为只能往大的方向换），所以我们无视 1。这个时候 2 又变成了最小，我们再去用同样的道理去换 2。直到所有数字都到达正确的位置上。

通过上面的例子，我们不难看出，对于确定的 A 和 B，我们有且只有一个固定的替换方式，来使得 A 变成 B。

而我们要问的是 B 的种数，那我们就需要确认，这个替换过程中，是否一直都满足 $i \leq j \leq i + s$。

这里有一个结论：**必须有 $s \geq \max (A_i - B_i)$**。

结合上面的标准化后的例子很容易理解：首先 $A_1 - 1$ 是肯定要的。而我们的替换过程一直是在让 $i$ 和此时的 $A_i$ 替换，因此 $A_i - i$ 是必要的。

而如果 $A_i < i$，那这个 $A_i$ 肯定会被前面某个 $A_k \geq k$ 的 $A_k$ 替换。而由于 k 在 i 前面，所以显然有 $A_k - k > A_k - i$，这个点不考虑也罢。因此，$s \geq \max (A_i - B_i)$ 的要求是充分且必要的。

接下来考虑计数的问题。有了这个结论之后，问题就变得简单了不少。我们的问题转化为求满足 $B_i \geq A_i - s$ 的排列数量。

我们把没有确定对应的 $A_i$ 弄出来排个序，然后我们从小到大枚举缺的 $B_i$，这样就保证了后面枚举到的 $b$ 的可选位置范围比前一个大。每个 $B_i$ 的可选位置数乘起来就是我们要的答案了。

具体看代码。

## 代码

```c++
#include <cstdio>
#include <algorithm>
using namespace std;
typedef long long ll;
const ll mod = 998244353;
const int N = 2e5 + 5;
int n, A[N], B[N], s, C[N], cc, D[N], cd; bool vis[N];
int main() {
    int T; scanf("%d", &T);
    while(T--) {
        scanf("%d%d", &n, &s);
        for(int i = 1; i <= n; i++) scanf("%d", A + i), vis[i] = 0;
        cc = cd = 0;
        for(int i = 1; i <= n; i++) {
            scanf("%d", B + i);
            if(B[i] == -1) C[++cc] = A[i];
            else vis[B[i]] = 1;
        }
        bool fail = 0;
        for(int i = 1; i <= n && !fail; i++) {
            if(B[i] != -1) {
                if(A[i] - B[i] > s) fail = 1;
                continue;
            }
        }
        if(fail) {
            puts("0"); 
            continue;
        }
        sort(C + 1, C + cc + 1);
        for(int i = 1; i <= n; i++) {
            if(!vis[i]) D[++cd] = i;
        }
        int p = 0; ll ans = 1;
        for(int i = 1; i <= cd; i++) {
            while(p < cc && D[i] >= C[p + 1] - s) p++;
            if(p - i + 1 <= 0) {
                ans = 0;
                break;
            }
            ans = ans * (ll)(p - i + 1) % mod;
        }
        printf("%lld\n", ans);
    }
    return 0;
}
```