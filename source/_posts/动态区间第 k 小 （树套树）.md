---
title: 动态区间第 k 小 （树套树）
author: LawrenceSivan
avatar: https://cdn.jsdelivr.net/gh/LawrenceSivan/cdn@master/pictures/avatar.jpg
authorLink: 'https://lawrencesivan.github.io/'
comments: true
mathjax: true
date: 2021-06-30 19:14:32
authorAbout:
authorDesc:
categories:
tags:
keywords: 数据结构,树套树,主席树,区间第 k 大
description: 区间第 k 大问题小总结以及动态区间第 k 大的树套树（树状数组，线段树）解法
photos: https://i.loli.net/2021/06/30/CNtkfjQndr1EyFH.jpg
---

# P2617 Dynamic Rankings

## 前言

这是一道已经咕了很久的题。

第一次想写是因为 luan 讲了这道题。

直到她又出现在了我的智能推荐里。

我知道我不能再等了！

~~我要去和她表白！！~~

## 类题总结

关于各种第 k 大问题肯定已经见过很多了。

反正各种奇奇怪怪的解法都是有的。

比较主流的有：

### 静态整体第 k 大（小）

这玩意有两种解法。

首先是显而易见的 `std::sort`.

```cpp
sort(a+1,a+1+n);
//reverse(a+1,a+1+n);
cout<<a[k]<<endl;
```

~~生怕你不会写这玩意~~

时间复杂度 $\text{O(nlogn)}$。

之后这玩意是存在 $\text{O(n)}$ 做法的。

考虑魔改快排。

每次选择基准值的时候记录一些大于基准值的个数 $cnt$ ，之后根据 $cnt$ 选择一半进入，就可以在平均线性之间内统计出第 k 大（小）。

```cpp
int getk(int *v, int l, int r, int k){
    if(l<r){
        int i=l,j=l;
        for(;j<r;j++){
            if(v[j]<v[r]){
                swap(v[i++],v[j]);
            }
        }
        swap(v[i],v[r]);
        if(k==i)  return v[i];
        else if(k<i)  return getk(v,l,i-1,k);
        else return getk(v,i+1,r,k);
    }
    else return v[l];
}
```

### 动态整体第 k 大（小）

权值线段树，插入直接在树上插入，然后查询也直接查就行,像个桶一样。

值域小一点的话直接建树就行了，大一点就动态开点。

```cpp
namespace SegmentTree{

    struct SegmentTree{
        int v;
        #define ls rt<<1
        #define rs rt<<1|1
    }st[maxn<<2]; 

    inline void push_up(int rt){
        st[rt].v=st[ls].v+st[rs].v;
    }

    void modify(int rt,int l,int r,int pos,int val){
        if(l==r){
            st[rt].v+=val;
            return;
        }

        int mid=(l+r)>>1;
        if(pos<=mid)modify(ls,l,mid,pos,val);
        else modify(rs,mid+1,r,pos,val);

        push_up(rt);
    }

    int ask(int rt,int l,int r,int val){//查询第 k 大
        if(l==r)return l;

        int mid=(l+r)>>1;
        if(val<=st[rs].v)ask(rs,mid+1,r,val);
        else ask(ls,l,mid,val-st[rs].v);
    }
}
```


复杂度 $\text{O(nlogn)}$


### 静态区间第 k 大（小）

也就是主席树板子题了。

具体看这个就行：[p3834 可持久化线段树模板（主席树）](https://www.luogu.com.cn/blog/LawrenceSivan/p3834-ke-chi-jiu-hua-xian-duan-shu-mu-ban-zhu-xi-shu-post)

### 动态区间第 k 大（小）

就是本题了。

考虑修改操作。

由于常规的主席树是依靠前缀和的思想来求解区间第 k 大的，于是可以想到求解动态区间第 k 大首要问题是如何维护前缀和的问题。

对于维护前缀和，我们通常有如下几种方式：

~~暴力~~，树状数组，线段树。

于是有一个思路：加一个数据结构来维护前缀和。

加上去的数据结构放在什么位置呢？

由于主席树需要前缀和来作差，于是可以发现树状数组是要套在主席树外面的。

于是明白树状数组每个节点都维护一个线段树的根节点。

分开考虑修改以及查询操作。

#### 修改操作

树状数组每次单点修改复杂度 $\text{O(logn)}$ ，对应 $\text{O(logn)}$ 棵权值线段树会受到影响，又因为每次修改需要对应到线段树内，线段树内每次又会有 $\text{O(logn)}$ 个节点需要修改。于是单次复杂度 $\text{O}(log^2n)$

每次修改需要用树状数组定位到需要修改的线段树的根节点位置，之后进行修改即可：

```cpp
void modify(int &rt,int l,int r,int pos,int val){
    if(!rt)rt=++node;
    
    if(l==r){
        st[rt].v+=val;
        return;
    }

    int mid=(l+r)>>1;
    if(pos<=mid)modify(ls(rt),l,mid,pos,val);
    else modify(rs(rt),mid+1,r,pos,val);

    push_up(rt);
}

void update(int pos,int x,int val){
    for(re int i=pos;i<=n;i+=lowbit(i)){//处理涉及到的 log(n) 个根节点，就是对应的主席树
        modify(root[i],1,tmp,x,val);//线段树上修改log(n) 个节点
    }
}

int main() {
    ...

    for(re int i=1;i<=m;i++){
        if(q[i].op){//modify
            update(q[i].pos,a[q[i].pos],-1);//把原数删去
            a[q[i].pos]=q[i].val;//换成现在要的数
            update(q[i].pos,q[i].val,1);//在树上修改
        }
    }

}
```

#### 查询操作

对于查询操作，依然是要用到前缀和。

由于使用了树状数组维护前缀和，于是我们直接在树状数组上取出对应的 $\text{O(logn)}$ 个根节点，也就是对应的主席树根节点，取的时候直接体现作差就行。

之后就和普通的主席树查询区间第 k 大一样，根据作差得到的值直接递归进入左子树或者右子树即可。

复杂度 $\text{O}(log^2n)$

```cpp
int query(int l,int r,int k){
    if(l==r)return l;

    int mid=(l+r)>>1;
    int xx=0;
    for(re int i=1;i<=cnt1;i++)xx-=st[ls(ct1[i])].v;//作差得到cnt值
    for(re int i=1;i<=cnt2;i++)xx+=st[ls(ct2[i])].v;
    
    if(xx>=k){//左递归，连带着log(n)棵主席树一起进入左儿子
        for(re int i=1;i<=cnt1;i++)ct1[i]=ls(ct1[i]);
        for(re int i=1;i<=cnt2;i++)ct2[i]=ls(ct2[i]);
        
        return query(l,mid,k);
    }

    else {////右递归，连带着log(n)棵主席树一起进入右儿子
        for(re int i=1;i<=cnt1;i++)ct1[i]=rs(ct1[i]);
        for(re int i=1;i<=cnt2;i++)ct2[i]=rs(ct2[i]);
        
        return query(mid+1,r,k-xx);
    }
}

inline void located(int l,int r){//取出对应的log(n)个线段树的根节点。
    cnt1=cnt2=0;
    for(re int i=l-1;i;i-=lowbit(i))ct1[++cnt1]=root[i];//取l-1和r，用于作差
    for(re int i=r;i;i-=lowbit(i))ct2[++cnt2]=root[i];  
}

int main() {
    ...

    for(re int i=1;i<=m;i++){
        else {
            located(q[i].l,q[i].r);
            ans=b[query(1,tmp,q[i].k)];
            printf("%d\n",ans);
        }
    }

}

```

#### 复杂度分析

查询操作 $\text{O}(log^2n)$，修改操作 $\text{O}(log^2n)$，总体复杂度 $\text{O}(mlog^2n)$

由于动态开点，所以空间复杂度就是对应的查询和修改所需要的空间。

于是 空间复杂度和时间复杂度可以认为是同阶的。

线段树大小开 $400$ 倍就够了。

具体实现的时候建议封一个 `namespace` ，看着也好看调着也爽。

似乎代码没有想象的那么长？

#### CODE:

```cpp
//#define LawrenceSivan

#include <bits/stdc++.h>
using namespace std;

typedef long long ll;
typedef unsigned long long ull;
#define INF 0x3f3f3f3f
#define re register
const int maxn=1e5+5;

int n,m,ans;

char op;

int a[maxn],b[maxn<<1],tot,tmp;

int ct1[maxn],ct2[maxn],cnt1,cnt2;

struct QUERY{
    int l,r,k,op;
    int pos,val;
}q[maxn];

namespace SegmentTree{
    inline void Discretization(){
        sort(b+1,b+1+tot);
        tmp=unique(b+1,b+1+tot)-b-1;
        for(re int i=1;i<=n;i++)a[i]=lower_bound(b+1,b+1+tmp,a[i])-b;
        for(re int i=1;i<=m;i++){
            if(q[i].op==1)q[i].val=lower_bound(b+1,b+1+tmp,q[i].val)-b;
        }
    }

    struct SegmentTree{
        int lc,rc,v;
        #define ls(x) st[x].lc
        #define rs(x) st[x].rc
    }st[maxn*400];

    int node,root[maxn];

    inline void push_up(int rt){
        st[rt].v=st[ls(rt)].v+st[rs(rt)].v;
    }

    void modify(int &rt,int l,int r,int pos,int val){
        if(!rt)rt=++node;
        
        if(l==r){
            st[rt].v+=val;
            return;
        }

        int mid=(l+r)>>1;
        if(pos<=mid)modify(ls(rt),l,mid,pos,val);
        else modify(rs(rt),mid+1,r,pos,val);

        push_up(rt);
    }

    int query(int l,int r,int k){
        if(l==r)return l;

        int mid=(l+r)>>1;
        int xx=0;
        for(re int i=1;i<=cnt1;i++)xx-=st[ls(ct1[i])].v;
        for(re int i=1;i<=cnt2;i++)xx+=st[ls(ct2[i])].v;
        
        if(xx>=k){
            for(re int i=1;i<=cnt1;i++)ct1[i]=ls(ct1[i]);
            for(re int i=1;i<=cnt2;i++)ct2[i]=ls(ct2[i]);
            
            return query(l,mid,k);
        }

        else {
            for(re int i=1;i<=cnt1;i++)ct1[i]=rs(ct1[i]);
            for(re int i=1;i<=cnt2;i++)ct2[i]=rs(ct2[i]);
            
            return query(mid+1,r,k-xx);
        }
    }

}

using namespace SegmentTree;

namespace BIT{
    #define lowbit(x) (x&-x)

    void update(int pos,int x,int val){
        for(re int i=pos;i<=n;i+=lowbit(i)){
            modify(root[i],1,tmp,x,val);
        }
    }

    inline void located(int l,int r){
        cnt1=cnt2=0;
        for(re int i=l-1;i;i-=lowbit(i))ct1[++cnt1]=root[i];
        for(re int i=r;i;i-=lowbit(i))ct2[++cnt2]=root[i];  
    }

}

using namespace BIT;

template <typename T>
inline void read(T &x){
    T f=1;char ch=getchar();
    while (!isdigit(ch)) {if(ch=='-')f=-1;ch=getchar();}
    while (isdigit(ch)){x=x*10+(ch^48);ch=getchar();}
    x*=f;
}

int main() {
#ifdef LawrenceSivan
    freopen("aa.in", "r", stdin);
    freopen("aa.out", "w", stdout);
#endif
    read(n);read(m);
    for(re int i=1;i<=n;i++){
        read(a[i]);
        b[++tot]=a[i];
    }

    for(re int i=1;i<=m;i++){
        //cin>>op;
        scanf(" %c",&op);
        if(op=='C'){
            q[i].op=1,read(q[i].pos),read(q[i].val),b[++tot]=q[i].val;
        }
        else q[i].op=0,read(q[i].l),read(q[i].r),read(q[i].k);
    }

    Discretization();

    for(re int i=1;i<=n;i++){
        update(i,a[i],1);
    }

    for(re int i=1;i<=m;i++){
        if(q[i].op){
            update(q[i].pos,a[q[i].pos],-1);
            a[q[i].pos]=q[i].val;
            update(q[i].pos,q[i].val,1);
        }
        else {
            located(q[i].l,q[i].r);
            ans=b[query(1,tmp,q[i].k)];
            printf("%d\n",ans);
        }
    }



    return 0;
}
```
