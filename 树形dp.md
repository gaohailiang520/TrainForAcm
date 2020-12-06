# 树形dp

背景知识：

由于用邻接矩阵相连的话会造成空间的浪费，所以大部分（数据范围大）时候使用邻接表建边

![img](https://bkimg.cdn.bcebos.com/pic/4e4a20a4462309f7bfc89bc4780e0cf3d6cad644?x-bce-process=image/watermark,image_d2F0ZXIvYmFpa2U4MA==,g_7,xp_5,yp_5)



### P1352 没有上司的舞会

```cpp
#include<iostream>
#include<algorithm>
#include<cstring>
#include<cstdio>
#include<cmath>
using namespace std;
int f[5][6000];
int n,a,b,root;
int ne[12000],po[12000],ru[12000];
void dp(int x)
{
    for(int i = po[x];i;i = ne[i])
    {
        dp(i);
        f[1][x] += f[0][i];//当前这个节点如果选择了，那么子节点不能选
        f[0][x] += max(f[1][i],f[0][i]);//当前这个节点不选，字节点可选可不选
    }
}
int main(){
    scanf("%d",&n);
    for(int i = 1;i <= n;i ++)
     scanf("%d",&f[1][i]);
    for(int i = 1;i < n;i ++) 
     {
         scanf("%d%d",&b,&a);
         ru[b] ++;
         //ne里面存放的是？po里里面存放的是？
         //ne里面存放的是兄弟节点，po里面存放的是孩子节点的起点
         ne[b] = po[a];
         po[a] = b;
     }
    for(int i = 1;i <= n;i ++)
	 if(ru[i] == 0) 
	   {
	        root = i;//树的根
	        break;
	   } 
    dp(root);   
    printf("%d",max(f[1][root],f[0][root]));
    return 0;
}
```

### P2015 二叉苹果树

树形dp+01背包

f(i, j) 表示以 i 为节点的根保留 k 条边的最大值接下来对每一个节点分类讨论，共三种情况：

1. 全选左子树 
2. 全选右子树 
3. 左右子树都有一部分

为了方便，我们设左儿子为 ls，右儿子为 rs，连接左儿子的边为 le，连接右儿子的边为 re
$$
f(i, j) = max \{f(ls, j − 1) + le, f(rs, j − 1) + re, max_{0≤k≤j−2}
\{f(ls, k) + f(rs, j − 2 − k)\} + le + re\}
$$

*处理某个之前，前面的都已经处理过了*

第二篇应该是没问题的，后面花时间看看

[P2015 二叉苹果树 - 洛谷 | 计算机科学教育新生态 (luogu.com.cn)](https://www.luogu.com.cn/problem/solution/P2015)


```cpp
#include<bits/stdc++.h>
using namespace std;
int son[105][105],f[105][105];
int n,m,w[105][105],cnt[105],vis[105];

void dfs(int k)
{
	vis[k]=1;// 每次循环时，给父结点一个标记，防止无限循环 
	for(int i=1;i<=cnt[k];i++)
	{
		int ny=son[k][i];// 给 ny 赋上 k 结点的第 i 个 子结点 
		if(vis[ny]==1)continue;// 如果 ny 是 k 的 父结点，直接跳过循环 
		vis[ny]=1;
		dfs(ny);// 以  ny 为 父结点 进行搜索 
		for(int j=m;j>=1;j--) //逆序枚举当前可以选的树枝根数 
			for(int g=j-1;g>=0;g--) //逆序枚举留给 子结点 的树枝根数 
			{
                //那排除了自己的了吗
				f[k][j]=max(f[k][j],f[ny][g]+f[k][j-g-1]+w[k][ny]);
				// f[ny][g] 表示以 ny 为 子结点 可以选 g 条边
				// f[k][j-g-1] 表示剩余给 k 结点的 兄弟结点 j-g-1 条边
				// w[k][ny] 表示 k 结点和 ny 结点之间的苹果数  
			}
	}
	return;
}

int main()
{
	cin>>n>>m;
	for(int i=1;i<n;i++)
	{
		int x,y,z;
		scanf("%d%d%d",&x,&y,&z);
        //cnt[x]表示子节点的隔宿
		w[x][y]=w[y][x]=z;//因为不知道父结点和子结点分别是谁，所以同时存值 
		son[x][++cnt[x]]=y;// 每输入一次当前结点，该节点的儿子数就 + 1 
		son[y][++cnt[y]]=x;
	}
	dfs(1);
	cout<<f[1][m]<<endl;//输出一 1 为根，选了 m 条边的树 
}
```

###  P1122最大子树和

状态：$f[u]$表示以u为根且包含u的最大权连通块

状态转移方程：$f[u]=max(0,f[v]);(v为u的孩子)$表示孩子可以选，可以不选

```cpp
#include<bits/stdc++.h>
using namespace std;
struct node
{
    int next,to;
}tree[100000];  //存图，蒟蒻不会stl
int n,a[100000],head[100000],cnt,f[100000],ans; //f[u] 表示以u为根且包含u的最大权联通块
void add(int x,int y) 
{
    tree[++cnt].next=head[x];
    tree[cnt].to=y;
    head[x]=cnt;
}
void dfs(int u,int fa)//u为当前节点,fa为u的爸爸
{
    f[u]=a[u];  //先给f[u]赋初值，就是u本身的美观指数
    for(int i=head[u];i;i=tree[i].next) //找儿子
    {
        int v=tree[i].to;  
        if(v!=fa)  //之前加的双向边，可能跑回去
        {
            dfs(v,u);  //继续向下找
            f[u]+=max(0,f[v]);  //状态转移
        }
    }
    ans=max(ans,f[u]); //更新ans
}
int main()
{
    scanf("%d",&n);
    for(int i=1;i<=n;i++)
        scanf("%d",&a[i]);
    for(int i=1,x,y;i<n;i++)
    {
        scanf("%d%d",&x,&y);
        add(x,y);add(y,x);  //注意加双向边
    }
    dfs(1,0);  
    printf("%d",ans);
}
```



### P2014 [CTSC1997]选课

没太懂难点，也没太看懂

这道题的意思是每本书要想选择一门课，必须要先学会它的必修课，所以这就形成了一种依赖行为，即选择一门课必须要选择必修课。那么他又说要选择的价值最大，这就要用到树形背包的知识了。 树形背包的基本代码形式（即上面的树形背包类）

```cpp
/*
设dp[i][j]表示选择以i为根的子树中j个节点。
u代表当前根节点，tot代表其选择的节点的总额。
*/
void dfs(int u,int tot)
{
	for(int i=head[x];i;i=e[i].next)
	{
		int v=e[i].to;
		for(int k=0;k<tot;k++)//这里k从o开始到tot-1，因为v的子树可以选择的节点是u的子树的节点数减一
			dp[v][k]=dp[u][k]+val[u];
		dfs(v,tot-1)
		for(int k=1;k<=tot;k++)
			dp[u][k]=max(dp[u][k],dp[v][k-1]);//这里是把子树的值赋给了根节点，因为u选择k个点v只能选择k-1个点。
	}
}
```

然后这就是树形背包的基本形式，基本就是这样做 代码

```cpp
#include<iostream>
#include<algorithm>
#include<queue>
#include<cstdio>
#include<cstring>
using namespace std;

int n,m;
struct edge
{
    int next,to;
}e[1000];
int rt,head[1000],tot,val[1000],dp[1000][1000];
void add(int x,int y)
{
    e[++tot].next=head[x];
    head[x]=tot;
    e[tot].to=y;
}
void dfs(int u,int t)
{
    if (t<=0) return ;
    for (int i=head[u]; i; i=e[i].next)
    {
        int v = e[i].to;
        for (int k=0; k<t; ++k) 
            dp[v][k] = dp[u][k]+val[v];
        dfs(v,t-1);
        for (int k=1; k<=t; ++k) 
            dp[u][k] = max(dp[u][k],dp[v][k-1]);
    }
}
int main()
{
    scanf("%d%d",&n,&m);
    for(int i=1;i<=n;i++)
    {
        int a;
        scanf("%d%d",&a,&val[i]);
        if(a)
          add(a,i);
        if(!a)add(0,i);
    }
    dfs(0,m);
    printf("%d",dp[0][m]);
}
```