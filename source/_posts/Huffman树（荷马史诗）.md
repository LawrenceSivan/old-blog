---
title: Huffman树（荷马史诗）
author: LawrenceSivan
avatar: https://gcore.jsdelivr.net/gh/LawrenceSivan/cdn@master/pictures/avatar.jpg
authorLink: 'https://lawrencesivan.github.io/'
comments: true
mathjax: true
date: 2021-02-05 07:39:37
authorAbout:
authorDesc:
categories:
tags:
keywords: 基本算法,贪心
description: Huffman编码与Huffman树
photos: https://images.pexels.com/photos/1183021/pexels-photo-1183021.jpeg?auto=compress&cs=tinysrgb&dpr=2&h=650&w=940
---

hi 

晚上好啊

今天讲了基本算法（基本听不懂的算法）——

~~又被学长虐了呢~~~~~

终于在下课前学长讲了一道能写得出来的题，

为了庆祝，

我们来写一篇博客。

## $Huffman$树

首先，我们来考虑这样一个问题：

构造一棵包含 $n$ 个叶子节点的 $k$ 叉树，其中第 $i$ 个叶子节点带有权值 $w_i$，要求最小化 $\sum w_i * l_i$，其中 $l_i$ 表示第 $i$ 个叶子节点到根节点的距离。

该问题的解被称为 $k$ 叉 $Huffman$ 树（哈夫曼树）

为了最小化 $\sum w_i * l_i$，应该让权值大的叶子节点的深度尽量小（这应该十分显然了）。

当 $k==2$ 时，我们就可以维护一个小根堆（出门左转合并果子）

对于 $k>2 $的 $k$ 叉 $Huffman$ 树的求解，直观的想法是在贪心的基础上，改为每次取出最小的 $k$ 个权值。

然鹅，经过仔细思考，可以发现：

如果在执行最后一轮循环时，堆的大小在 $2-k$ 之间（也就是说，不足以取出 $k$ 个），那么整棵$Huffman$ 树的根的子节点数就小于 $k$。这显然不是最优解——我们取 $Huffman$ 树中任意一个深度最大的节点，把他改为树根的子节点，就会使 $\sum w_i * l_i$ 变小。

因此，我们在执行贪心步骤之前，首先要补充一些额外的权值为 $0$ 的叶节点，使得叶节点的个数 $n$ 满足$(n-1)mod(k-1)==0$.

也就是说，我们让子节点不足k的情况发生在最底层，而不是在根结点处，在 $(n-1)mod(k-1)==0$ 时，执行“每次从堆中取出最小的 $k$ 个权值”的贪心思路是正确的。




------------
那么，我们现在来看 $p2168$ 荷马史诗

其实看了上面的，题目自然就很显然了

~~啊淦为什么我也这么爱说显然了~~

那我们把单词出现的次数作为权值，然后直接构造 $k$ 叉 $Huffman$ 树，对于每个节点的第 $k$ 个分支，分别标记为 $0-k-1$；

下面我们需要一点跟 $trie$ 有关的东西

如果把这个 $Huffman $树看作是一颗 $trie$，那么就得到了总长度最小的编码方式——单词 $i$ 的编码就是从根节点到叶节点 $i$ 上各个字符相连（如果没学过 $trie$ 建议去看一看，反正也不难，一举两得）。

那么如何保证一个单词不是另一个单词的前缀呢？

我们只需要保证，单词的结尾是叶子节点就好了啊（这是 $trie $的性质，如果一个串是另一个串的前缀的话，那么这个串的结尾一定还可以向后延伸，也就是可以经变长变为另一个更长的单词。）

如何保证 $s_i$ 最短呢？

只需要在构造 $Huffman$ 树的时候，对于权值相同的节，我们优先合并当前深度小（也就是合并次数小的）即可。

CODE：

```cpp
#include"bits/stdc++.h"
using namespace std;
#define ll long long
struct node{//结构体，这个是节点，里面有两个域，分别是权值和当前深度（也就代表着合并次数）
	ll w,h;
	node(ll w,ll h):w(w),h(h){}//构造函数赋初值
	bool operator < (const node &a)const{//我们重载小于号，使之成为一个优先按照权值大小排序，深度（即合并次数）为第二关键字的小根堆
		return w==a.w?h>a.h:w>a.w;
	}
};
ll ans;//答案
priority_queue <node> q;//小根堆
int main(){
	ios::sync_with_stdio(false);
	ll n,k,ans=0;
	cin>>n>>k;//可以想一下，k进制代表每个数字都是不超过k-1的，那么我们可以联想到，这不就是k叉Huffman的k吗？
	for(int i=1;i<=n;i++){
		ll w;
		cin>>w;//我们把出现次数看成权值
		q.push((node){w,1});
	}
	while((q.size()-1)%(k-1)!=0)q.push((node){0,1});//判断是不是要补上空节点
	while(q.size()>=k){//合并操作
		ll h=-1,w=0;
		for(int i=1;i<=k;i//取出最小的k个
			node t=q.top();
			q.pop();
			h=max(h,t.h);
			w+=t.w;
		}
		ans+=w;
		q.push((node){w,h+1});
	}
	cout<<ans<<endl<<q.top().h-1<<endl;
	
	return 0;
}
```

