---
title: NTT快速数论变换学习笔记
author: LawrenceSivan
avatar: https://cdn.jsdelivr.net/gh/LawrenceSivan/cdn@master/pictures/avatar.jpg
authorLink: 'https://lawrencesivan.github.io/'
authorAbout: 'I was so much older then, I’m younger than that now.'
authorDesc: 'I was so much older then, I’m younger than that now.'
comments: true
mathjax: true
date: 2021-08-15 18:40:56
categories:
tags:
keywords: 多项式,数论,NTT,卷积
description:  NTT 快速数论变换学习笔记
photos: https://i.loli.net/2021/08/15/MPjLzExfW65viqn.jpg
---

# NTT 快速数论变换学习笔记

## 前言

向多项式迈出的勇敢第二步！

~~虽然也是基础的东西~~	

写了这个博客真的是对数论内容有了很多新的认识，也学到了很多新东西。

甚至解决了很多之前遗留的没有解决的问题。

- 以下内容均在剩余系意义下讨论。

## 阶

### 定义

$a$ 在模 $p$ 意义下的阶：

最小的整数 $k$ 使得 $a^k\equiv1\ \pmod p$。

记为 $δ_p(a)$ 。

更通俗的说法是幂的最小循环节。

### 定理

上限是 $\varphi(p)$。

模 $p$ 的阶如果存在，那么一定是 $\varphi(p)$ 的约数。

证明：

- 定理 1：若 $p>1$ 且 $a\perp p$ ，且 $a^n \equiv 1\ \pmod p$ ，$n>0$ ，则 $δ_p(a)|n$
- 定理 2：结合欧拉定理：当 $a,p \in N^+$，并且 $a \perp p$ ，那么有 $a^{\varphi(p)}\equiv 1\ \pmod p$，并结合定理 1 可知 $δ_p(a)|\varphi(p)$。

证毕。

或者以另外一种方式来说：

如果这个数有阶，那么这个数可以表示为 $g^k$ 的形式。

能够在 $\varphi(p)$ 大小的环上得到一个 $\dfrac{\varphi(p)}{\gcd(k,\varphi(p))}$ 大小的环，这就是它的阶，这显然是 $\varphi(p)$ 的约数。

可以从这种角度来理解定理 1 和定理 2。 

对于 $a \not\perp p$ 的情况，阶认为是 $∞$ 或者不存在。

## 原根

### 定义

满足 $δ_p(g)=\varphi(p)$ 的 $g$ （达到上界），我们称之为 $p$ 的原根。

### 定理

- 只有形如 $2,4,p^x,2p^x$ 才有原根，其中 $p$ 是奇质数。

一旦求出了其中一个原根，就可以构造出其他的原根。

所有的 $g^k$ 当 $(k \perp \varphi(p))$ 时，恰好能够构造出 $\varphi(\varphi(p))$ 个，且不重不漏。

- 定理 1：若 $g$ 是 $m$ 的原根，则

$$g,g^2...g^{\varphi(p)}$$

各数模 $p$ 的最小剩余，恰好是小于 $p$ 并且与 $p$ 互素的 $\varphi(p)$ 个正整数的一个排列。

- 定理 2：每一个质数 $p$ 都有 $\varphi(p-1)$ 个原根。事实上每一个数 $x$ 都有 $\varphi(\varphi(x)$ 个原根（如果他有原根的话）

推论

- 若 $d|(p-1)$ ，则 $x^d \equiv 1\ \pmod p$ 恰好有 $d$ 个解
- 若 $p$ 是质数，且 $d|(p-1)$ ，则阶为 $d$ 的最小剩余 $\bmod p$ 的个数为 $\varphi(d)$。

### 求法

由于原根一般不会很大（可以证明 $n$ 的最小原根在 $\mathcal{O(n^{0.25})}$ 级别），因此我们暴力枚举+验证的方法。

上面我们提到，模 $p$ 的阶如果存在，那么一定是 $\varphi(p)$ 的约数。（详情看上文中 ：阶/定理）

于是计算阶时，只需要验证 $\varphi(p)$ 的约数即可。

其实并不需要算出阶是多少,只需要查看阶是否是 $\varphi(p)$ 即可。

首先将 $\varphi(p)$ 分解质因数：

$$\varphi(p)=p_1^{c_1}p_2^{c_2}p_3^{c_3}...p_x^{c_x}$$

注意到 $a^k \equiv 1\ \pmod p$ ，那么 $a^{ck}\equiv 1\ \pmod p$。

可以取出 $ \varphi(p)$ 的所有质因数 $p_1,p_2,p_3...p_x$ ，要判断 $g$ 是否是原根，只需要判断 $g^{\frac{\varphi(p)}{p_i}}\not\equiv 1\ \pmod p$ 恒成立，那么 $g$ 就是 $p$ 的一个原根。

这样可以保证不重不漏地覆盖所有其他约数的倍数，并且达不到 $\varphi(p)$ 本身。

数据不大的时候枚举所有 $\varphi(p)$ 的因子(除本身)也可。

##  原根与单位根

经过对 FFT 的学习，我们发现由于 FFT 需要使用单位根在复平面上进行系列运算，所以决定它必须要使用浮点数，从而引发精度误差。 

但是在复数域 $\C$ 中，单位根是唯一满足要求的一类数。

在剩余系意义下我们可以找到单位根的类似物——**原根**

首先考虑一下单位根的独特性质：

1. $\omega_n^k=(\omega_n^1)^k$

2. $\omega_n^{0\sim(n-1)}$ 必须互不相同

   否则点值重复，插值不正确。

3. $\omega_n^k=\omega_n^{k\mod n}$

   $n$ 是偶数的时候，可以根据对称引理得出求和引理。

通过以上几条性质可以得出，$\omega_n^1$ 在模意义下的阶恰好是 $n$。

4. $\omega_{2n}^{2k}=\omega_n^k$

   等价于 $(\omega_{2n})^2=\omega_n$

回顾原根的定义，对于一个**素数** $p$ ，如果 $g$ 的阶达到了 $p-1$ 的上界，则称 $g$ 是 $p$ 的原根。

显然 $n$ 不一定等于 $p-1$ ，所以原根并不能直接使用。

上面我们提到阶的两个定理以及另一种证明方法。

$g^k$ 的阶数恰为 $\dfrac{p-1}{\gcd(k,p-1)}$，这个数仍然是 $p-1$ 的约数。

所以当 $n\not|\ (p-1)$ 时，找不到阶恰好为 $n$ 的数。

当 $n|(p-1)$ 时，$g^\frac{(p-1)}{n}$ 的阶恰好是 $\dfrac{p-1}{\gcd(\frac{p-1}{n},p-1)}=n$

$(\omega_{2n})^2=(g^{\frac{p-1}{2n}})^2=g^{\frac{p-1}{n}}=(\omega_n^1)^k=\omega_n$

所以 $g^{\frac{p-1}{n}}$ 满足了上述所有要求！

至于 $n|(p-1)$ 的要求，事实上在分治过程中，$n$ 往往会被补成 $2$ 的次幂，于是我们只需要保证 $p-1$ 包含很多个因子 $2$ 即可。

常见的 NTT 模数有 

$$P=998244353=479∗2^{21}+1,g=3$$

$$P=1004535809=7∗17∗2^{23}+1,g=3$$

## get_NTT!

```cpp
//#define LawrenceSivan

#include <bits/stdc++.h>
using namespace std;

typedef long long ll;
typedef unsigned long long ull;

#define INF 0x3f3f3f3f
#define re register

const int maxn=1e7+5;
const int mod=998244353;
const int G=3;

ll qpow(ll a,ll b){
    ll res=1;
    for(;b;b>>=1,a=a*a%mod){
        if(b&1)res=res*a%mod;
    }
    return res%mod;
}

const ll invG=qpow(G,mod-2);

ll f[maxn],p[maxn];

int trans[maxn];

int n,m,invn;

inline void NTT(ll *f,bool op){
    for(re int i=0;i<n;i++){
        if(i<trans[i])swap(f[i],f[trans[i]]);
    }
    for(re int p=2;p<=n;p<<=1){
        int len=p>>1;
        int tG=qpow(op?G:invG,(mod-1)/p);
        for(re int k=0;k<n;k+=p){
            ll buf=1;
            for(re int l=k;l<k+len;l++){
                ll tmp=buf*f[l+len]%mod;
                f[l+len]=f[l]-tmp;
                if(f[l+len]<0)f[l+len]+=mod;
                f[l]=f[l]+tmp;
                if(f[l]>mod)f[l]-=mod;
                buf=buf*tG%mod;
            }
        }
    }
}

namespace IO{
    template<typename T>
    inline void read(T &x){
        x=0;T f=1;char ch=getchar();
        while (!isdigit(ch)) {if(ch=='-')f=-1;ch=getchar();}
        while (isdigit(ch)){x=x*10+(ch^48);ch=getchar();}
        x*=f;
    }

    template <typename T, typename... Args>
    inline void read(T& t, Args&... args) {
        read(t); read(args...);
    }

    template<typename T>
    void write(T x){
        if(x<0)putchar('-'),x=-x;
        if(x>9)write(x/10);
        putchar(x%10+'0');
    }

    template<typename T,typename... Args>
    void write(T t,Args... args){
        write(t);putchar(' ');write(args...);
    }
}

using namespace IO;

signed main() {
#ifdef LawrenceSivan
    freopen("aa.in","r", stdin);
    freopen("aa.out","w", stdout);
#endif
    read(n,m);n++,m++;
    for(re int i=0;i<n;i++)read(f[i]);
    for(re int i=0;i<m;i++)read(p[i]);
    for(m+=n,n=1;n<m;n<<=1);
    for(re int i=0;i<n;i++){
        trans[i]=(trans[i>>1]>>1)|((i&1)?n>>1:0);
    }
    NTT(f,1),NTT(p,1);
    for(re int i=0;i<n;i++)f[i]=f[i]*p[i]%mod;
    NTT(f,0);invn=qpow(n,mod-2);
    for(re int i=0;i<m-1;i++)write((f[i]*invn)%mod),putchar(' ');
    puts("");

    return 0;
}
```



## 参考资料

[NTT与多项式全家桶 by command_block](https://www.luogu.com.cn/blog/command-block/ntt-yu-duo-xiang-shi-quan-jia-tong)

[阶 和 原根 by _duadua](https://blog.csdn.net/a27038/article/details/77203892)

