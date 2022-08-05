---
title: 快速莫比乌斯&沃尔什变换(FMT&FWT)学习笔记
author: LawrenceSivan
avatar: https://gcore.jsdelivr.net/gh/LawrenceSivan/cdn@master/pictures/avatar.jpg
authorLink: 'https://lawrencesivan.github.io/'
authorAbout: 'I was so much older then, I’m younger than that now.'
authorDesc: 'I was so much older then, I’m younger than that now.'
comments: true
mathjax: true
abstract: This blog is encrypted.
message: You must enter the password to read.
date: 2021-08-25 20:45:47
categories:
tags: 快速莫比乌斯变换,快速沃尔什变换,FWT,FMT
keywords:
description: 这是一个咕咕咕一万年的学习笔记
photos: https://i.loli.net/2021/08/25/Vk1wgbxQWcTPhJB.jpg
password:
---

# 快速莫比乌斯&沃尔什变换(FMT&FWT)学习笔记

## 前言

就是这个混蛋玩意我学了~~两个晚上~~15天才搞定 /fn

~~其实是因为一直在摸鱼，导致......~~

好了于是今晚决定不摸！

（来自未来：这个 sb 又咕咕咕了

updated on 2021.9.9：被点击哥断电了，气愤

updated on 2021.9.9：半个小时后：重写的时候发现了一些错误，感谢点击哥。

updated on 2021.9.9 16:56 ,这只鸽子现在终于写完了

## 位运算卷积

之前提到了 FFT（快速傅里叶变换）、NTT（快速数论变换），他们都是对于加法卷积(多项式乘法)的快速变换。

回忆一下卷积的一般形式是：

$$\sum\limits_{i\oplus j=k}{A[i]B[j]}$$

根据上式我们可以给出位运算卷积的定义：

将第 $i$ 项与第 $j$ 项的乘积贡献到第 $i\oplus j$ 项，其中 $\oplus$ 是一种位运算。

## 思路

大致思路与 FFT 一致，首先进行正变换，之后逐位相乘，最后经过逆变换得出答案。

## 或卷积（FWT_or）

要求：

$$C[i]=\sum\limits_{j|k=i}{A[j]B[k]}$$

### 正变换 （FWT）

有个显然的性质是 $j|i=i,k|i=i \rightarrow(j|k)|i=i$。

于是可以构造：

$$FWT[A]_i=\sum\limits_{j|i=i}A[j]$$

意义是下标的子集对应位置之和。

$j|i=i$ 表示 $j$ 是 $i$ 的子集。

于是有：

$$FWT[A]_i\times FWT[B]_i=\left(\sum\limits_{j|i=i}{A[j]}\right)\left(\sum\limits_{k|i=i}{B[k]}\right)$$

$$=\sum\limits_{j|i=i}\sum\limits_{k|i=i}{A[j]B[k]}$$

$$=\sum\limits_{(j|k)|i=i}{A[j]B[k]}$$

$$=\sum\limits_{x|i=i}\sum\limits_{j|k=x}{A[j]B[k]}$$

$$=\sum\limits_{x|i=i}C[x]$$

$$=FWT[C]_i$$

依照这个思路，我们就完成了正变换。

直接做显然是 $\mathcal{O(n^2)}$ 的。

考虑如何快速实现这个过程：

回忆 FFT 的实现过程，我们通过分治降低了复杂度。

于是将整个区间分成两部分，进行分治。

对于一个长度为 $2^n$ 的多项式，我们使用 $A_0$ 表示前 $2^{n-1}$ 项，$A_1$ 表示后 $2^{n-1}$ 项。

可以发现，对于 $A_0$ ，他的下标最高位一定是 $0$，于是他的子集只能是自己的子集。

对于 $A_1$ 由于最高位是 $1$，所以它的子集除了包括自己的部分，还要包括 $A_0$ 的部分。

于是可以写出以下式子：

$$FWT[A]= \begin{cases}{\text{merge}(FWT[A_0],FWT[A_0]+FWT[A_1]),(n>0)}\\{A,(n=0)} \end{cases} $$

其中， $\text{merge}$ 表示拼接，$+$ 表示对应位置相加。

这就是分治手段。

事实上是高维前缀和.

### 逆变换 (IFWT)

满足 $A=IFWT(FWT(A))$。

#### 感性理解&做法

嗯，写这个主要是为了方便 ~~（逃~~

把正变换的符号换一下就行了。

由 

$${\text{merge}(FWT[A_0],FWT[A_0]+FWT[A_1]),(n>0)}$$ 

得：

$$IFWT(FWT(A))=merge(IFWT(FWT(A_0)),IFWT(FWT(A_1))-IFWT(FWT(A_0)))$$

简单一点就是：

$$A=merge(A_0,A_1-A_0)$$

感性理解就是前缀和与差分的关系。

实际上是高维前缀差分.

#### 理性理解&证明

证明：

现在我们已知 $FWT[A]_0,FWT[A]_1$ ，要求出 $A_0,A_1$。

根据上面提到的 $A_0$ 的子集只包括自己的那一部分，得到：

$$FWT[A]_0=FWT[A_0]$$

所以:

$$A_0=IFWT(FWT(A_0))=IFWT(FWT(A)_0)$$

其实我认为证明到这里就应该结束了。

但是他居然还有一部分，而且我认为越扯越远.

依据上面提到的 $A_1$ 还要包括 $A_0$ 的那一部分，得到：

$$FWT[A]_1=FWT[A_0]+FWT[A_1]$$

$$FWT[A_1]=FWT[A]_1-FWT[A_0]=FWT[A]_1-FWT[A]_0$$

所以：

$$A_1=IFWT(FWT(A_1))=IFWT(FWT[A]_1-FWT[A]_0)$$

之后合并：

$$A=merge(IFWT(FWT(A)_0),IFWT(FWT[A]_1-FWT[A]_0))$$

~~似乎感性理解已经完全够用了~~

### CODE:

```cpp
inline void FWT_or(int *a,int op){
    for(re int i=1;i<n;i<<=1){
        for(re int p=i<<1,j=0;j<n;j+=p){
            for(re int k=0;k<i;k++){
                (a[i+j+k]+=a[j+k]*op+mod)%=mod;
            }
        }
    }
}
```

## 与卷积（FWT_and）

要求：

$$C[i]=\sum\limits_{j \& k=i}{A[j]B[k]}$$

### 正变换 （FWT）

有了前面的构造经验，我们可以类似地得出以下构造方式：

$$FWT[A]_i=\sum\limits_{i|j=j}A[j]$$

意义依旧是是下标的子集对应位置之和。

$i|j=j$ 表示 $i$ 是 $j$ 的子集。

证明：

$$FWT[A]_i\times FWT[B]_i=\left(\sum\limits_{i|j=j}{A[j]}\right)\left(\sum\limits_{i|k=k}{B[k]}\right)$$

$$=\sum\limits_{i|j=j}\sum\limits_{i|k=k}A[j]B[k]$$

$$=\sum\limits_{i|(j\&k)=(j\&k)}A[j]B[k]$$

$$=\sum\limits_{i|x=x}\sum\limits_{j\&k=x}{A[j]B[k]}$$

$$=\sum\limits_{i|x=x}C[x]$$

$$=FWT[C]_i$$

可以发现与卷积和或卷积十分相似，前者是后面对前面有贡献，后者是前面对后面有贡献。

类比或卷积即可得到递归公式：

$$FWT[A]= \begin{cases}{\text{merge}(FWT[A_0]+FWT[A_1]，FWT[A_1]),(n>0)}\\{A,(n=0)} \end{cases} $$

证明：

依旧考虑分治：

对于一个长度为 $2^n$ 的多项式，我们使用 $A_0$ 表示前 $2^{n-1}$ 项，$A_1$ 表示后 $2^{n-1}$ 项。

可以发现，由于按位与的性质是集合只会变小不会变大，所以分成左右两个部分之后 $A_1$ 与 $A_0$ 按位与得到的结果只能是首位为 $0$ ，这么说来后面会变成前面的子集，而前面只会包含自己，所以后面的贡献要加到前面去。

实际上是高维后缀和。

### 逆变换 (IFWT)

满足 $A=IFWT(FWT(A))$。

#### 感性理解&做法

依旧是把正变换的符号换一下就行了。

由 

$${\text{merge}(FWT[A_0]+FWT[A_1]+FWT[A_1]),(n>0)}$$ 

得：

$$IFWT(FWT(A))=merge(IFWT(FWT(A_0))-IFWT(FWT(A_1)),IFWT(FWT(A_1))$$

简单一点就是：

$$A=merge(A_0-A_1,A_1)$$

实际上是高维后缀差分.

#### 理性理解&证明

是真的不太清楚这玩意有啥用，咕了。

### CODE:

```cpp
inline void FWT_and(int *a,int op){
	for(re int i=1;i<n;i<<=1){
		for(re int p=i<<1,j=0;j<n;j+=p){
			for(re int k=0;k<i;k++){
				(a[j+k]+=a[i+j+k]*op+mod)%=mod;
			}
		}
	}
}
```

## 异或卷积（FWT_xor）

要求：

$$C[i]=\sum\limits_{j \oplus k=i}{A[j]B[k]}$$

### 正变换 （FWT）

参照上面的构造经验，我们需要构造出变换满足：

$$FWT(C)_x=FWT(A)_xFWT(B)_x$$

考虑到 FWT 是线性变换，所以可以做出以下构造：

$$FWT(F)_x=\sum\limits_{i=0}^{n}g(x,i)F_i$$

其中 $g$ 是一个系数。

于是需要做到：

$$\sum\limits_{i=0}^{n}g(x,i)C_i=\left(\sum\limits_{j=0}^{n}g(x,j)A_j\right)\left(\sum\limits_{k=0}^{n}g(x,k)B_k\right)$$

代入 $C_i$ 得到：

$$\left(\sum\limits_{i=0}^{n}g(x,i)\right)\left(\sum\limits_{j \oplus k=i}{A[j]B[k]}\right)=\left(\sum\limits_{j=0}^{n}g(x,j)A_j\right)\left(\sum\limits_{k=0}^{n}g(x,k)B_k\right)$$

化简整理可得：

$$\sum\limits_{j=0}^{n}\sum\limits_{k=0}^{n}g(x,j \oplus k)A[j]B[k]=\sum\limits_{j=0}^{n}\sum\limits_{k=0}^{n}g(x,j)g(x,k)A[j]B[k]$$

于是只需要令：

$$g(x,j \oplus k)=g(x,j)g(x,k)$$

即可。

接下来的手法就有些奇妙了。

考虑异或操作之后 $i,j$ 的什么属性会把他们和 $i \oplus j$ 联系起来。

有个结论是在异或前后 $1$ 的个数的奇偶性不会发生变化。

具体就是异或前 $i,j$ 的 $1$ 的个数的和与 $i \oplus j$ 的奇偶性相同。

证明过程是按位考虑的。

分两种情况：

1. 两个数对应位相同，如果都是 $1$ ，那么 $1$ 的个数减少 $2$ ，奇偶性不变；如果都是 $0$ ，那么 $1$ 的个数不变化。
2. 两个数不同，只能是一个 $1$ 一个 $0$ ，所以 $1$ 的数量不变。

证毕；

形式化的说法是:

>  $ \rm{popcount(a)+popcount(b)\equiv popcount(a\ xor\ b)\pmod 2}$

于是我们可以构造：

$$g(x,i)=(-1)^{|i \cap x|}$$

每一项是二进制表示的集合

$i \cap x$ 实际就是取出了 $i$ 中 $x$ 是 $1$ 的位。

如此构造满足了：

$$g(x,j \oplus k)=g(x,j)g(x,k)$$

证明：

由：

$$g(x,i)=(-1)^{|i \cap x|}$$

得：

$$g(x,j \oplus k)=g(x,j)g(x,k)$$

$$=(-1)^{|(j \oplus k) \cap x|}=(-1)^{|j \cap x|}(-1)^{|k \cap x|}$$

$$=(-1)^{|(j \oplus k) \cap x|}=(-1)^{|j \cap x|+|k \cap x|}$$

其中 $i \cap x$ 表示取出了 $i$ 中 $x$ 是 $1$ 的位。

其中 $|i \cap x|$ 表示取出了 $i$ 中 $x$ 是 $1$ 的位的个数。

根据上面的结论，$|(j \oplus k) \cap x|\equiv|j \cap x|+|k \cap x|\pmod 2$

所以两式确实相等。

证毕。

于是我们得到了 FWT_xor 的变换方式：

$$FWT(F)_x=\sum\limits_{i=0}^{n}(-1)^{|i \cap x|}F_i$$

或许你看着这样的比较顺眼：

$$FWT(F)_x=\sum\limits_{i=0}^{n}(-1)^{|i \& x|}F_i$$

也可以写成：

$$FWT(F)_x=\sum\limits_{\text{c}(x\&i)\pmod 2=0}F[i]-\sum\limits_{\text{c}(x\&k)\pmod 2=1}F[k]$$

其中 $c$ 代表 $\text{popcount}$. 

这里似乎有一个关于底数问题的讨论？

~~但是我觉得似乎比较显然，就放个链接略过吧~~

[关于底数问题的讨论](https://moyujiang.blog.luogu.org/solution-p4717)

每一个位置对于 $FWT(F)_x$ 都有贡献，贡献的正负由集合大小的奇偶性决定。

递归式是这样的：

$$FWT[A]= \begin{cases}{\text{merge}(FWT[A_0]+FWT[A_1]，FWT[A_0]-FWT[A_1]),(n>0)}\\{A,(n=0)} \end{cases} $$

快速变换还是基于分治增量：

对于新加入的一位，分两种情况考虑，把当前集合也分成选或不选，取放右边，不取放左边。

1. 取，如果与取的状态合并，集合大小不变；若和不取的状态合并，那么集合大小-1，所以贡献取反。
2. 不取，与取或不取状态取并，集合大小都不会变。

于是取会累加到取，累减到不取。

不取同时累加到取或者不取。

### 逆变换 (IFWT)

满足 $A=IFWT(FWT(A))$。

#### 感性理解&做法

如果方便记忆的话，那么直接 $\times inv2$ 即可。

感性一些的做法是直接移项两式和差什么的就好了。

$$IFWT(FWT(A))=merge\left(\dfrac{IFWT(FWT(A_0))+IFWT(FWT(A_1))}{2},\dfrac{IFWT(FWT(A_0)-IFWT(FWT(A_1)}{2}\right)$$

即：

$$A=\text{merge}\left(\dfrac{A_0+A_1}{2},\dfrac{A_0-A_1}{2}\right)$$

### CODE:

```cpp
inline void FWT_xor(int *a,int op){
	for(re int i=1;i<n;i<<=1){
		for(re int p=i<<1,j=0;j<n;j+=p){
			for(re int k=0;k<i;k++){
				int X=a[j+k],Y=a[i+j+k];
				a[j+k]=(X+Y)%mod;
				a[i+j+k]=(X+mod-Y)%mod;
				(a[j+k]*=op)%=mod,(a[i+j+k]*=op)%=mod;
			}
		}
	}
}
```

似乎代码还挺~~好背~~简单的？

## 全部代码：

```cpp
//#define LawrenceSivan

#include <bits/stdc++.h>
using namespace std;

typedef long long ll;
typedef unsigned long long ull;

#define INF 0x3f3f3f3f
#define re register
#define int ll

namespace IO{
	template<typename T>
	inline void read(T &x){
		x=0;T f=1;char ch=getchar();
		while (!isdigit(ch)) {if(ch=='-')f=-1;ch=getchar();}
		while (isdigit(ch)){x=x*10+(ch^48);ch=getchar();}
		x*=f;
	}

	template <typename T, typename... Args>
	inline void read(T& t, Args&... args){
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

const int maxn=1<<17|1;
const int mod=998244353;

int n,inv2=(mod+1)>>1;

int A[maxn],B[maxn],a[maxn],b[maxn];

inline void copy(){
	memcpy(a,A,sizeof(A));
	memcpy(b,B,sizeof(B));
}

inline void mul(){
	for(re int i=0;i<n;i++){
		a[i]=(a[i]*b[i])%mod;
	}
}

inline void print(){
	for(re int i=0;i<n;i++){
		write(a[i]),putchar(' ');
	}
	puts("");
}

inline void FWT_or(int *a,int op){
	for(re int i=1;i<n;i<<=1){
		for(re int p=i<<1,j=0;j<n;j+=p){
			for(re int k=0;k<i;k++){
				(a[i+j+k]+=a[j+k]*op+mod)%=mod;
			}
		}
	}
}

inline void FWT_and(int *a,int op){
	for(re int i=1;i<n;i<<=1){
		for(re int p=i<<1,j=0;j<n;j+=p){
			for(re int k=0;k<i;k++){
				(a[j+k]+=a[i+j+k]*op+mod)%=mod;
			}
		}
	}
}

inline void FWT_xor(int *a,int op){
	for(re int i=1;i<n;i<<=1){
		for(re int p=i<<1,j=0;j<n;j+=p){
			for(re int k=0;k<i;k++){
				int X=a[j+k],Y=a[i+j+k];
				a[j+k]=(X+Y)%mod;
				a[i+j+k]=(X+mod-Y)%mod;
				(a[j+k]*=op)%=mod,(a[i+j+k]*=op)%=mod;
			}
		}
	}
}

signed main(){
#ifdef LawrenceSivan
	freopen("aa.in","r", stdin);
	freopen("aa.out","w", stdout);
#endif
	read(n);n=1<<n;
	for(re int i=0;i<n;i++)read(A[i]);
	for(re int i=0;i<n;i++)read(B[i]);
	copy();FWT_or(a,1),FWT_or(b,1),mul(),FWT_or(a,-1),print();
	copy();FWT_and(a,1),FWT_and(b,1),mul(),FWT_and(a,-1),print();
	copy();FWT_xor(a,1),FWT_xor(b,1),mul(),FWT_xor(a,inv2),print();


	return 0;
}
```

## 参考资料

[题解 快速莫比乌斯/沃尔什变换 (FMT/FWT) - 摸鱼酱的博客 - 洛谷博客 (luogu.org)](https://moyujiang.blog.luogu.org/solution-p4717)

[题解 P4717 【模板】快速沃尔什变换 - xht37 的洛谷博客 - 洛谷博客 (luogu.com.cn)](https://www.luogu.com.cn/blog/xht37/solution-p4717)

[快速沃尔什变换（FWT）详详详解_a_forever_dream的博客-CSDN博客_fwt符号意思](https://blog.csdn.net/a_forever_dream/article/details/105110089)

[真正理解快速沃尔什变换/快速莫比乌斯变换(FWT|FMT) (已完结) (leanote.com)](http://blog.leanote.com/post/rockdu/TX20)

[FWT(快速沃尔什变换)零基础详解qaq（ACM/OI） - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/41867199)

[从线性变换的角度看待 FWT - Gauss's blog - 洛谷博客 (luogu.org)](https://gauss0320.blog.luogu.org/zong-xian-xing-bian-huan-di-jiao-du-kan-dai-fwt)
