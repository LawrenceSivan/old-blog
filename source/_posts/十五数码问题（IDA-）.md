---
title: 十五数码问题（IDA*）
author: LawrenceSivan
avatar: https://cdn.jsdelivr.net/gh/LawrenceSivan/cdn@master/pictures/avatar.jpg
authorLink: 'https://lawrencesivan.github.io/'
comments: true
mathjax: true
date: 2021-04-15 12:15:46
authorAbout:
authorDesc:
categories:
tags:
keywords: 启发式搜索,搜索,IDA*
description: 启发式搜索IDA*
photos: /images/cover/2.jpeg
---

~~这个题可真是快调死我了~~

题面很简单，就是十五个数字的华容道，然后问最少几步可以还原成初始状态
（貌似是人工智能经典问题？

有了八数码问题的经验，我们知道这玩意肯定是要各种玄学做法（比如什么要人命的康托展开双向广搜$A*$迭代加深$IDA*$  $bulabula$~

康托展开就算了吧，那玩意真心懒得写，而且十五数码问题用这种东西貌似不合适（其实是不会

$A*$写起来太麻烦，于是写写$IDA*$罢,貌似这题双向广搜会快一些？

其实$IDA*$也没有很慢好吧 ~~（不开$O_2$根本过不了~~

对于估价函数，我们可以考虑曼哈顿距离，估价函数的值等同于当前状态下所有非 $0$ 的数字到目标位置的曼哈顿距离之和。

而且对于$15$数码问题，我们可以得出以下结论：十五数码和八数码判断是否有解的方法不同，八数码$0$的移动不影响其余$7$个数字逆序数的奇偶性，而十五数码$0$的左右移动不影响其余$15$个数逆序数的奇偶性 （顺序不变），但上下移动改变奇偶（移动三次），加上$0$的话$16$个数逆序数左右改变（移动一次），上下也改变（移动$7$次），需要注意每次移动$0$的距离奇偶性也改变（$0$到目标位置的曼哈顿距离不是加1就是减一），所以$16$个数逆序数与$0$的距离之和$s$的奇偶性不因$0$的滑动而改变，初始时$s$是奇数，所以只有$s$是奇数的状态才是可到达的.

于是我们在搜之前先判一下有没有解：

```cpp
inline bool check(){//check it is the ans or not
    int cnt=0,tot=0;
    for (int i=1;i<=4;++i){
    	for (int j=1;j<=4;++j){
    		puz[tot]=tmp[i][j];
    		tot++;
		}
	}
    for (int i=0;i<16;++i){
        if (puz[i]==0)
            cnt+=3-i/4;
        else{
            for (int j=0;j<i;++j){
            	if (puz[j] && puz[j]>puz[i]){
            		cnt++;
				}
			}
        }
    }
    return !(cnt&1);
}
```

之后我们还需要加入一个剪枝：永远不要走和上一步相反的路；

大概就这些

```cpp
//#define LawrenceSivan

#include<bits/stdc++.h>
using namespace std;

typedef long long ll;
#define re register

int x,y,cnt; 
int tmp[5][5];
int puz[16]; 

const int goal[5][5]={
	{0,0,0,0,0},
	{0,1,2,3,4},
	{0,5,6,7,8},
	{0,9,10,11,12},
	{0,13,14,15,0}
};

const int dx[4]={0,1,-1,0};
const int dy[4]={1,0,0,-1};

const int px[16]={4,1,1,1,1,2,2,2,2,3,3,3,3,4,4,4};
const int py[16]={4,1,2,3,4,1,2,3,4,1,2,3,4,1,2,3};

inline int mabs(int a){
	return a>0?a:-a;
}

inline bool check(){//check it is the ans or not
    int cnt=0,tot=0;
    for (int i=1;i<=4;++i){
    	for (int j=1;j<=4;++j){
    		puz[tot]=tmp[i][j];
    		tot++;
		}
	}
    for (int i=0;i<16;++i){
        if (puz[i]==0)
            cnt+=3-i/4;
        else{
            for (int j=0;j<i;++j){
            	if (puz[j] && puz[j]>puz[i]){
            		cnt++;
				}
			}
        }
    }
    return !(cnt&1);
}

inline int evaluate(){//judge it can get the ans or not
    int res=0;
    for(int i=1;i<=4;i++){
    	for(int j=1;j<=4;j++){
    		if(tmp[i][j]==0)continue;
       	 	res+=mabs(i-px[tmp[i][j]])+mabs(j-py[tmp[i][j]]);
    	}
    }
    return res;
} 

inline int getway(int i){
	if(dx[i]==-1)return 2;
	if(dx[i]==1)return 1;
	if(dy[i]==-1)return 4;
	if(dy[i]==1)return 3;
}

inline bool test(int i,int pre){
	if(dx[i]==-1&&pre==1)return true;
	if(dx[i]==1&&pre==2)return true;
	if(dy[i]==-1&&pre==3)return true;
	if(dy[i]==1&&pre==4)return true;
	
	return false;
}

bool IDA_star(int step,int x,int y,int pre){//do it
	int eva=evaluate();
	if(!eva)return true;
	
	if(step+eva>cnt)return false;
	
	for(int i=0;i<4;i++){ 
        int xx=x+dx[i];
        int yy=y+dy[i];
        if(xx>4||xx<1||yy>4||yy<1||test(i,pre))continue;
        swap(tmp[xx][yy],tmp[x][y]);
        if(IDA_star(step+1,xx,yy,getway(i)))return true;		
		swap(tmp[xx][yy],tmp[x][y]);
    }
    return false;
}

inline int read(){
	int x=0,f=1;char ch=getchar();
	while(!isdigit(ch)){if(ch=='-')f=-1;ch=getchar();}
	while(isdigit(ch)){x=x*10+(ch^48);ch=getchar();}
	return x*f;
}

int main(){
#ifdef LawrenceSivan
	freopen("puzzle.in","r",stdin);
	freopen("puzzle.out","w",stdout);
#endif
	for(re int i=1;i<=4;i++){
		for(re int j=1;j<=4;j++){
			tmp[i][j]=read();
			if(tmp[i][j]==0)x=i,y=j;
		}
	}
	/*for(re int i=1;i<=4;i++){
		for(re int j=1;j<=4;j++){
			cout<<tmp[i][j]<<" ";
		}
		cout<<endl;
	}*/
	
	if(!evaluate()){//if the begin is the end
        printf("%d\n",0);
        return 0;
    }
    
    if(!check()){//if the begin is the end
        printf("No\n");
        return 0;
    }
    
    while(++cnt){
    	if(IDA_star(0,x,y,0)){
            printf("%d\n",cnt);
            return 0;
        }
        if(cnt>50){
        	printf("No\n");
			return 0;	
		}
	}
    
    return 0;
}

```
