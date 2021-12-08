# **CHD-ACM 2019 Summer Traninging Personal Summary**

## 一. **What I Learned**

### 1.  搜索相关

####    1．A*

A∗算法是一种带有估价函数的优先队列bfs,即A∗是一种启发式搜索算法。

定义起点  s，终点 t 。

从起点（初始状态）开始的距离函数 g(x) 。

到终点（最终状态）的距离函数 h(x), h*(x) 。

定义每个点的估价函数 f(x) = g(x) + h(x) 。

A*算法每次从 **优先队列** 中取出一个 f 最小的，然后更新相邻的状态。

如果 h<h* ，则 A*算法能找到最优解。

上述条件下，如果 h **满足三角形不等式，则 A\*算法不会将重复结点加入队列** 。

**例题：第k短路**

**POJ 2449**

建图的时候建一个正向的一个反向的，反向的图利用dij求终点到每个点的最短路，即为搜索的估价函数。

注意的地方：

st==en 的时候必须k++ 因为题目要求必须走过路径。

~~~C++
#include <iostream>
#include <cstdio>
#include <cstring>
#include <algorithm>
#include <queue>
using namespace std;

#define inf 0x7ffffff
#define MAXN 1010
#define MAXM 100005

struct Edge
{
    int u;//反向图的终止边
    int v;//终止点
    int c;//边权值
    int next;//下一个节点    
    int next1;//反向边下一个节点
    Edge(){}
    Edge(int u,int v,int c):u(u),v(v),c(c){}
}p[MAXM];
int head[MAXN];//链表头
int head1[MAXN];//反向
int e; //邻接表中边总数
int st,en,k;//起点，终点，第k短路
int n,m;
int dis[MAXN];//dis[v]表示的是终点到v的距离，即估价函数g
bool vis[MAXN];

struct pro
{
    int v,c;//v是终点,c是起点到v的距离
    pro(){}
    pro(int v,int c):v(v),c(c){}
    bool operator < (const pro& a) const 
    {
        return c+dis[v] > a.c+dis[a.v];//最小值优先队列
    }
};

void clear()//初始化
{
    memset(head,-1,sizeof(head));
    memset(head1,-1,sizeof(head1));
    e=0;
}
void addEdge(int u,int v,int c)//加边
{
    p[e]=Edge(u,v,c);
    p[e].next1=head1[v];head1[v]=e;
    p[e].next=head[u];head[u]=e++;
}

priority_queue<pro> que;
void dij(int src)//求估价函数
{
    memset(vis,0,sizeof(vis));
    for(int i=1;i<=n;i++)
        dis[i]=inf;
    dis[src]=0;
    while(!que.empty())
        que.pop();
    que.push(pro(src,0));
    while(!que.empty())
    {
        pro cur = que.top();
        que.pop();
        if(vis[cur.v])
            continue;
        vis[cur.v]=1;
        for(int i=head1[cur.v];i+1;i=p[i].next1)
        {
            if(dis[p[i].u] > dis[cur.v] + p[i].c)
            {
                dis[p[i].u] = dis[cur.v] + p[i].c;
                que.push(pro(p[i].u,0));
            }
        }
    }
}

int a_star(int src)
{
    while(!que.empty())
        que.pop();
    que.push(pro(src,0));
    while(!que.empty())
    {
        pro cur = que.top();
        que.pop();
        if(cur.v==en)
        {
            if(k>1)//相当于求k次最短路
                k--;
            else
                return cur.c;
        }

        for(int i=head[cur.v];i+1;i=p[i].next)//将所有与u相连接的点都加入队列
            que.push(pro(p[i].v,cur.c+p[i].c));
    }
    return -1;
}

int main()
{
    int u,v,c;
    while(scanf("%d%d",&n,&m) != EOF)
    {
        clear();
        while(m--)
        {
            scanf("%d%d%d",&u,&v,&c);
            addEdge(u,v,c);
        }
        scanf("%d%d%d",&st,&en,&k);
        dij(en);
        if(dis[st]==inf)
        {
            puts("-1");
            continue;
        }
        if(st == en)
            k++;
        printf("%d\n",a_star(st));
    }
    return 0;
}
~~~



### 2.  动态规划

####    1．区间dp

区间类动态规划是线性动态规划的扩展，它在分阶段地划分问题时，与阶段中元素出现的顺序和由前一阶段的哪些元素合并而来由很大的关系。令状态 f(i,j) 表示将下标位置 i 到 j 的所有元素合并能获得的价值的最大值，那么 
$$
f(i,j)=max\{f(i,k)+f(k+1,j)+cost\}
$$
 cost  为将这两组元素合并起来的代价。

**合并** ：即将两个或多个部分进行整合，当然也可以反过来；

**特征** ：能将问题分解为能两两合并的形式；

**求解** ：对整个问题设最优值，枚举合并点，将问题分解为左右两个部分，最后合并两个部分的最优值得到原问题的最优值。

**状态转移方程**:
$$
f(i,j)=max\{f(i,k)+f(k+1,j)+\sum_{t=i}^ja_t\}
$$
sum_i  表示 a 数组的前缀和:
$$
f(i,j)=max\{f(i,k)+f(k+1,j)+sum_{j}-sum_{i-1}\}
$$
**poj1179**

~~~C++
#include <iostream>
#include <cstring>
#include <string>
#include <cstdio>
#include <set>
using namespace std;
const int N = 1e2+10;
int n;
int dpmin[N][N],dpmax[N][N],a[N],ans;
char b[N];

int main()
{
        cin >> n;
        for(int i = 1; i <= 2* n; i++){
                if(i % 2 == 1)  cin >> b[i/2 + 1],b[i/2 + 1 +n] = b[i/2 +1];
                else cin >> a[i/2],a[i/2 + n]=a[i/2];
        }

        memset(dpmin, 0x3f3f ,sizeof(dpmin) );
        memset(dpmax, 128, sizeof(dpmax));

        for(int i = 1; i <= 2*n; i++){
                dpmin[i][i] = a[i];
                dpmax[i][i] = a[i]; 
        }

        for(int len = 1; len <= n; len++)
                for(int l = 1; l <= 2*n - len ; l++){
                        int r = l + len;
                        for(int k = l; k<= r-1; k++ ){
                                if(b[k+1] == 't' ){
                                        dpmin[l][r] = min(dpmin[l][r], dpmin[l][k] + dpmin[k+1][r]);
                                        dpmax[l][r] = max(dpmax[l][r], dpmax[l][k] + dpmax[k+1][r]);
                                }
                                else {
                                        dpmin[l][r] = min(dpmin[l][r], dpmax[l][k] * dpmin[k+1][r]);
                                        dpmin[l][r] = min(dpmin[l][r], dpmin[l][k] * dpmax[k+1][r]);
                                        dpmin[l][r] = min(dpmin[l][r], dpmin[l][k] * dpmin[k+1][r]);
                                        dpmax[l][r] = max(dpmax[l][r],  dpmax[l][k] * dpmax[k+1][r]);
                                        dpmax[l][r] = max(dpmax[l][r],  dpmin[l][k] * dpmin[k+1][r]);
                                }
                        }
                }
        for(int l = 1; l <= n; l++)     ans = max(ans, dpmax[l][n+l-1]);
        cout<<ans<<endl;
        for(int i = 1; i <= n; i++ )
        if(ans == dpmax[i][n+i-1]) cout<<i<<" ";
}
~~~



####    2．状压dp

复习位运算内容

把集合转化为整数记录在dp状态中的算法

**POJ 2411**

~~~C++
#include <iostream>
#include <cstring>
#include <cmath>
#include <string>
#include <vector>
#include <algorithm>
#include <cstdio>
#include <queue>
#include <set>
typedef long long ll;
using namespace std;
const int maxn = 1e3 + 50;
const int INF = 2147483647;
int n, t, k,m,count;
ll f[12][1<<11];
bool s[1<<11];
int main()
{
	while(cin >> n >> m && n){
		for(int i = 0; i < 1 << m; i++){
			bool has = 0,cnt = 0;
			for(int j = 0; j < m; j++)
				if(i >> j & 1)	has |= cnt, cnt = 0;
				else cnt ^= 1;
			s[i] =has | cnt ? 0 : 1;
		}
		f[0][0] = 1;
		for(int i = 1; i <= n; i++){
			for(int j = 0; j < 1 << m; j++){
				f[i][j] = 0;
				for(int k = 0; k < 1 << m; k++){
					if((j & k) == 0 && s[j | k])
						f[i][j] += f[i-1][k];
				}
			}
		}
		cout << f[n][0]<<endl;
	}
	return 0;
}
~~~





####    3．树形dp

**阶段**：节点从深到浅的顺序（子树从小到大）

第一维常为节点编号

**没有上司的舞会**

~~~C++
#include <iostream>
#include <stack>
#include <string>
#include <cstring>
#include <algorithm>
#include <cstdio>
#include <vector>
#include <set>
using namespace std;
const int N = 1e4+50;
const int MOD = 1e9;
int v[N], dp[N][2], h[N];
int n,ans,root;
vector<int> son[N];

void dodp(int x)
{
        dp[x][1]=h[x];
        dp[x][0]=0;
        for(int i = 0; i < son[x].size(); i++){
                int y = son[x][i];
                dodp(y);
                dp[x][0] += max(dp[y][0], dp[y][1]);
                dp[x][1] += dp[y][0];
                }
}
int main()
{
        ios::sync_with_stdio(false);
        while(cin>>n){
        for(int i = 1; i <= n; i++) cin>>h[i];
        for(int i = 1; i <= n; i++){
                int x,y;
                cin>>x>>y;
                if(x==0&&y==0)break;
                v[x] = 1;
                son[y].push_back(x);
        }
        for(int i = 1; i <= n; i++){
                if(!v[i]){
                        root = i;
                        break;           
                }
        }
        dodp(root);
        cout << max(dp[root][0], dp[root][1]) << endl;
        }
}
~~~

树形+换根

**POJ 2196**

~~~c++
#include <iostream>
#include <stack>
#include <string>
#include <cstring>
#include <algorithm>
#include <cstdio>
#include <vector>
#include <set>
using namespace std;
const int MAXN=10010;
struct Node
{
    int to;
    int next;
    int len;
}edge[MAXN*2];
int head[MAXN];
int tol;
int maxn[MAXN];
int smaxn[MAXN];
int maxid[MAXN];
int smaxid[MAXN];

void init()
{
    tol=0;
    memset(head,-1,sizeof(head));
}

void add(int a,int b,int len)
{
    edge[tol].to=b;
    edge[tol].len=len;
    edge[tol].next=head[a];
    head[a]=tol++;
    edge[tol].to=a;
    edge[tol].len=len;
    edge[tol].next=head[b];
    head[b]=tol++;
}

void dfs1(int u,int p)
{
    maxn[u]=0;
    smaxn[u]=0;
    for(int i=head[u];i!=-1;i=edge[i].next)
    {
        int v=edge[i].to;
        if(v==p)continue;
        dfs1(v,u);
        if(smaxn[u]<maxn[v]+edge[i].len)
        {
            smaxn[u]=maxn[v]+edge[i].len;
            smaxid[u]=v;
            if(smaxn[u]>maxn[u])
            {
                swap(smaxn[u],maxn[u]);
                swap(smaxid[u],maxid[u]);
            }
        }
    }
}
void dfs2(int u,int p)
{
    for(int i=head[u];i!=-1;i=edge[i].next)
    {
        int v=edge[i].to;
        if(v==p)continue;
        if(v==maxid[u])
        {
            if(edge[i].len+smaxn[u]>smaxn[v])
            {

                smaxn[v]=edge[i].len+smaxn[u];
                smaxid[v]=u;
                if(smaxn[v]>maxn[v])
                {
                    swap(smaxn[v],maxn[v]);
                    swap(smaxid[v],maxid[v]);
                }
            }
        }
        else
        {
            if(edge[i].len+maxn[u]>smaxn[v])
            {
                smaxn[v]=edge[i].len+maxn[u];
                smaxid[v]=u;
                if(smaxn[v]>maxn[v])
                {
                    swap(smaxn[v],maxn[v]);
                    swap(maxid[v],smaxid[v]);
                }
            }
        }
        dfs2(v,u);
    }
}
int main()
{
    ios::sync_with_stdio;
    int n;
    int v,len;
    while(cin >> n)
    {
        init();
        for(int i=2;i<=n;i++)
        {
            scanf("%d%d",&v,&len);
            add(i,v,len);
        }
        dfs1(1,-1);


        dfs2(1,-1);
        for(int i=1;i<=n;i++)
          printf("%d\n",maxn[i]);
    }
    return 0;
}
~~~



### 3. 字符串

####    1．Hash

将string映射为int 每一个字符串代表一个数字 时间复杂度变为O(n+m)

~~~c++
ll hashs(char s[])//上述公式相当于把一个字符串看成一个b进制数 
{    
    int len=strlen(s);    
    ll ans=0;    
    for (int i=0;i<len;i++)        
        ans=(ans*base+(ll)s[i])%mod;    
    return ans; 
}
~~~



####    2．Trie

~~~c++
#include <bits/stdc++.h>
using namespace std;
const int N = 1e5+10;
int son[N][26] , cnt[N] , idx;
char str[N];

void Insert(char *str){
        int p = 0;
        for(int  i = 0; str[i]; i++){
                int u = str[i] - 'a';
                if(!son[p][u])  son[p][u] = ++ idx;
                p = son[p][u];
        }
        cnt[p] ++;
}

int Query(char *str){
        int p = 0;
        for(int i = 0; str[i]; i ++){
                int u = str[i] - 'a';
                if(!son[p][u])  return 0;
                p = son[p][u];
        }
        return cnt [p];
}
int main()
{
        int n;
        cin >> n;
        while (n -- ){
                char s[2];
                cin >> s >> str ;
                if(s[0] == 'I' )   Insert(str);
                else cout << Query(str)<<endl;
        }
}
~~~



####    3．最小表示法

~~~c++
int getmin(string s, int l){      //最小表示法
            int i=0,j=1,k=0;
    while(i<l&&j<l&&k<l)
    {
        if(s[(i+k)%l]==s[(j+k)%l])
            k++;
        else if(s[(i+k)%l]>s[(j+k)%l])
        {
            i = i+k+1;
            k = 0;
        }
        else 
        {
            j = j+k+1;
            k = 0;
        }
        if(i==j)
            j++;
    }
    return min(i,j);
}
~~~



####    4．kmp

**next** **数组求法**

~~~C++
next[1] = 0;
for(int i = 2,j = 0; i <=n; i++){
    while(j > 0 && a[i] != a[j+1]) j = next[j];
    if(a[i] == a[j+1])	j++;
    next[i] = j;
}
~~~

最小循环元

~~~C++
#include <iostream>
#include <cstring>
#include <cstdio>
using namespace std;
const int N= 1e6+10;
char a[N];
int next[N],n,t;

void getnext(){
        next[1]=0;
        for(int j=0,i=2;i<=n;i++){
                while(j > 0  && a[i] != a[j+1])         j=next[j];
                if(a[i] == a[j+1])      j++;
                next[i] = j;
        }
}

int main()
{
        while(cin>>n&&n){
                cin>>a+1;
                getnext();
                cout<<"Test case #"<<++t<<endl;
                for(int i = 2 ; i <=n; i++){
                        if(i%  (i - next[i]) == 0 && i / (i - next[i]) > 1)   
                                cout<< i << " " <<  i / ( i - next [i])<<endl;
                }
                cout<<endl;
        }

      //  system("pause");
	return 0;
}

~~~



####    5．AC自动机

~~~c++
#include <iostream>
#include <queue>
#include <cstring>
#include <cstdio>
using namespace std;
const int N = 1e6+10;
int tr[N][26] , fail[N] ,e[N], idx;
char s[N];

void Insert(char *s){
        int u=0;
        for(int i=1; s[i]; i ++) {
                int p = s[i] - 'a';
                if(! tr[u][p])  tr[u][p] = ++ idx;
                u = tr[u][p];
        }
        e[u]++;
}

queue<int> q;
void build(){
        for(int i = 0; i < 26; i ++){
                if(tr[0][i])    q.push(tr[0][i]);   
        }
        while(q.size()){
                int u = q.front();
                q.pop();
                for(int i = 0; i <  26; i ++){
                        if(tr[u][i])    fail[tr[u][i]] = tr[fail[u]][i], q.push(tr[u][i]);
                        else  tr[u][i] = tr[fail[u]][i];
                }
        }
}

int query(char *s){
        int res = 0, u =0;
        for(int i = 1; s[i]; i ++){
                int p = s[i] - 'a';
                u = tr[u][p];
                for(int j = u; j&&e[j] != -1; j = fail[j]){
                        res += e[j], e[j] = -1; 
                }
        }
        return res;
}

int main()
{
        int n;
        cin >> n;
        while(n --){
                cin >> s + 1 ;
                Insert(s);
        }
        build();
        cin >> s + 1;
        cout << query(s) << endl;
      // system("pause");
}
~~~



### 4. 数学部分

####    1．组合数学

##### （1）组合数

组合数性质 

1. $$ {C_n^m=Cm−nmCmn=Cmm−n}
    C_n^m=C_{n−1}^m+C_{n-1}^{m-1}
    $$

2. $$
    C_n^0+c_n^1+c_n^2+...+C_n^n=2^n
    $$

3. $$
    \sum_{i=0}^mC_n^iC_m^{m-i}=C_{m+n}^{m}(n>m)(分开取和一起取是一样的)
    $$

    

    
##### （2）多重集排列数

$$
\frac {n!}{n_1!n_2!...n_k!}
$$

##### （3）多重集组合数

n种不一样的球，每种球的个数是无限的，从中选k个出来组成一个多重集 （不考虑元素的顺序），产生的不同多重集的数量为：
$$
C_{n+k-1}^{k-1}
$$


##### （3）不相邻排列

1~n 这 n 个自然数中选k 个，这k 个数中任何两个数不相邻数的组合有:
$$
C_{n+k-1}^k
$$


##### (4)圆排列

$$
Q_n=(n-1)！
$$

部分圆排列：
$$
Q_n^r=\frac {A_n^r}{r}=\frac {n!}{r*(n-r)!}
$$


##### (5)组合数求法

- 递推公式O(n^2)

- 定义（预处理阶乘）

- 数字比较大阶乘会爆的话就会取模。然后除法用逆元来算，卢卡斯可以 更好的做这件事情

  **卢卡斯定理**
  $$
  C_n^k=C_{\frac kn}^{\frac kp}*C_{n\%p}^{n\%p}(mod\ p)
  $$
  



##### （6)卡特兰数

**引子**：给一个凸多边形，用n-3条不相交的对角线把它分成n-2个三角形， 求不同的方法数目
边 在最终的剖分种一定会恰好属于某个三角形 两条边将多边形划分为了一个k边形和一个 边形

#####    (7)斯特林数

#####    (8)康托展开

#####    (9)容斥原理

#### 2.概率与数学期望

#####  （1）全概率公式

若事件 A1,A2,...,An构成一个完备的事件且都有正概率，即对任意i,j,Ai交Aj=空集 且Ai的和为1 ，有
$$
P(B)=\sum_{i=1}^nP(A_i)P(B|A_i)
$$

#####  （2）贝叶斯公式

$$
P(B_i|A)=\frac {P(B_i)*p(A|B_i)}{\sum_{j=1}^nP(B_j)P(A|B_j)}
$$



##### （3）期望

**全期望公式** ： 
$$
E（Y）=E[E(Y|X)]
$$
。可由全概率公式证明。

**线性性质** ：对于任意两个随机事件 （不要求相互独立），有
$$
E(X+Y)=E(X)+E(Y)
$$


####    3．博弈论

##### bash博弈

**题目：**

  n个石子，每次取m个，无法取，则输

**思路:**

  1. n=0,先手必败
  2. **n!=0,n=(m+1)*k+s**
     - 先手赢：s!=0,  **n%(m+1)!=0**
     - 后手赢：s==0,**n%(m+1)==0**

**例题**

  1. #### hdu_2188
~~~C++
     #include <iostream>
     #include <iomanip>
     typedef long long ll;
     using namespace std;
     const int N=100000;
     const double pi=3.141592654;
     int main ()
     {
             int c,n,m;
             cin>>c;
             while(c--){
                     cin>>n>>m;
                     if(n%(m+1)!=0)  cout<<"first"<<endl;
                     else if(n%(m+1)==0)     cout<<"second"<<endl;
             }
            //system("pause");
     }
~~~

##### Nim博弈

**题目：**

  假设有n堆石子，每堆石子分别有a_1, a_2,…,a_n个,
  每次可以选择任意一堆且至少取1枚石子,
  甲乙两个玩家轮流取石子, 最后把石子取完的人获胜, 保证甲乙每一步的决策都是最优的, 甲为先手操作, 问甲胜还是乙胜。

**思路:**

  设 res= a_1^a_2^,…,^a_n 。

  若res = 0, 则先手必败, 反之必胜。

**例题**

~~~c++
        int n,a[N],res,ans;
         while(1){
                ans=0;
                res=0;
                cin>>n;
                if(n==0)        break;
                for(int i=0;i<n;i++){
                        cin>>a[i];
                        res^=a[i];
                }
                if(res==0)      cout<<"0"<<endl;
                else{
                for ( int i = 0; i < n; i++){
                        if((res^a[i])<a[i])       ans++;
                }
                cout<<ans<<endl;
                }
        }
~~~



##### 台阶Nim

**题目**：

•问题描述: 有一个n级台阶的楼梯, 每级台阶上有若干个石子, 其中第i级台阶上有a_i个石子(i≥1).

•两位玩家路轮流操作, 每次操作可以从任意一级台阶上拿若干个石子放到下一级台阶上(不能不拿)。

•已经拿到地面的石子不能再拿, 最后无法进行操作的人视为失败。

•问如果两人都采取最优策略, 先手是否必胜。

**思路**：

结论: res= a_1^a_3^a_5^,…,^a_n=0(当然这里的n是奇数)先手必败, 反之先手必胜。

即我们统计所有奇数部分的异或。

**POJ 1704**

每两个棋子作为一对，每对棋子之间的空位作为每堆石子的数量，在对手移动每对石子靠右的那一颗时，移动几位就相当于取几个石子，各堆的石子取尽，相当于不能再取石子

但是显然存在一些情况，因为题目规则并没有限制，所以如果我将每一对的棋子的左端棋子移动，就会导致堆中石子数增加
然而显然的是，若对手这么做要破坏棋局，我们可以将当前堆的右端棋子移动相同的步数，使堆中的棋子恢复到原来的数量，这样棋局就变的相当于没有改变

转化为台阶nim

~~~C++
        int t,a[N],n,m,res,ans;
        cin>>t;
         while(t--){
                ans=0;
                res=0;
                cin>>n;
                for(int i=1;i<=n;i++){
                        cin>>a[i];          
                }
                sort(a+1,a+n+1);
                for(int i=1;i<=n;i++){
                        if( ((i&1)&&(n&1)) || (!(i&1)&&!(n&1)) )
                        res^=a[i]-a[i-1]-1;     
                }
              //  cout<<res<<endl;
                if(!res)      cout<<"Bob will win"<<endl;
                else{
                cout<<"Georgia will win"<<endl;
                }
        }
~~~



##### Wythoff博弈(威佐夫博弈)

**题目：**

  两堆石子各有若干个, 两人轮流从一堆取至少一个石子或从两堆取同样多的物品, 最后一名取完石子者胜利。

**奇异局势**：a0=b0=0,a(k)是未在前面出现过的最小自然数,而 b(k)= a(k) + k，

**思路:**

  当两堆石子各有n和m个且不妨设n<m。

  当(m-n)  (√5+1)/2=n, 先手必败

**例题**

有两堆石子，数量任意，可以不同。游戏开始由两个人轮流取石子。游戏规定，每次有两种不同的取法，一是可以在任意的一堆中取走任意多的石子；二是可以在两堆中同时取走相同数量的石子。最后把石子全部取完者为胜者。

~~~C++
        int t,n,m,res,ans;
         while(cin>>n>>m){
                 if(n>m)        t=n,n=m,m=t;
                 //cout<< floor( (sqrt(5.0)+1)/2 )*(m-n)<<"  "<<n <<endl;
                 if(n == floor((( sqrt(5.0)+1)/2 )*(m-n)))     cout<<"0"<<endl;
                 else   cout<<"1"<<endl;
        }
~~~

hdu 2177

要求输出取一次石子后的剩下两堆石子的数量x和y, 分三种情况

~~~C++
#include <iostream>
#include <cmath>
using namespace std;
typedef long long ll;   
const int MAXN = 1e6+10, M = 30010, inf = 1e9;
const double alp = (sqrt(5)+1)/2;
int main() {
    int m, n, res, a, b;
    while(cin >> n >> m && n != 0 && m != 0) {
        if(int((m-n)*alp) == n) cout << "0" << endl;
        else {
            puts("1");
            if(n > m) swap(m, n);
            //求出两个数字的间距𝑑, 然后求出对应的奇异局势, 如果与两个数字差相同，我们就输出减去相同数字后得出的解。
            int res = m - n;
            a = res*alp, b = a + res;
            if(n - a == m - b) printf("%d %d\n", a, b);
			//求出以n为第一数字的奇异局势, 设第二数字为x, 若x<m, 表示可以在右边的石子中取一定的石子让后手面对奇异局势。
            int x = n;
            double b = x/alp;
            int y = n+b;
            if(m > y) printf("%d %d\n", x, y);
            //x>m, 则以m为第二数字, 从𝑛中拿走一定的石子变为第一数字。
            else{
                double x = n*1.0 / (alp);
                x = int(x);
                cout << x << " " << n << endl;
            }
        }
    }
}
~~~



##### Fibonacci博弈

**题意：**一堆石子有n个,两人轮流取.先取者第1次可以取任意多个，但不能全部取完。以后每次取的石子数不能超过上次取子数的2倍。取完者胜。给定n, 问先手必胜还是必败

**结论**: 当n为fibonacci数的时候, 先手必败

注：Fibonacci到50就超过2^31, 循环只用到50即可

~~~c++
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;   
const int MAXN = 1e6+10, M = 30010, inf = 1e9;
ll a[MAXN];
int main() {
    int n;
    a[1] = 1; a[2] = 1;
    for(int i = 3; i < 50; i++) {
        a[i] = a[i-1] + a[i-2];
    }
    while(cin >> n && n != 0) {
        int flag = 0;
        for(int i = 3; i < 50; i++) {
            if(n == a[i]) {
                flag = 1;break;
            }
        }
        if(flag) cout << "Second win" << endl;
        else cout << "First win" << endl;
    }
}
~~~



##### SG函数

有向无环图求Sg值

•mex运算: 定义mex(S)为不属于集合S的最小非负整数运算。

•例: S={1, 2, 3}, mex(s) = 0;

•SG函数: 设对于每个节点x, 设从x出发有k条有向边分别到达节点`y1` `y2`,…,`yk`,定义SG(x)函数为后继节点`y1` `y2`,…,`yk`的SG函数值构成的集合再执行mex运算的结果。

•特别的, 整个有向图G的SG函数被定义为有向图起点s的SG函数值, 即SG(G)=SG(s)

•有向图终点的SG函数为0。

•**结论**: 先手必败, 则该局面对应SG函数=0。反之必胜。

•**对于多个有向图**: 令`res=SG(s_1 )^SG(s_2 )^^… ^SG(s_n)`;

•若res=0, 则总局面必败。

•若res≠0,则总局面必胜。

**HDU** **1524**

~~~C++
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;   
const int MAXN = 1010, M = 30010, inf = 1e9;
int sg[MAXN]; //v[]统计到哪一个正整数了
struct Edge
{
	int to,next;
};
Edge edge[MAXN << 2];
int cnt = 0,n;
int head[MAXN];
void add(int u, int v)
{
	edge[cnt].to = v;
	edge[cnt].next = head[u];
	head[u] = cnt++;
}

int getsg(int u){   //求sg函数，深搜
    if (-1 != sg[u]) return sg[u];
    int v[MAXN];    //v[]的定义要写在里面，不然求出错误的值，可是为什么啊
    memset(v, 0, sizeof(v));
    for (int i = head[u]; -1 != i; i = edge[i].next) {
        v[getsg(edge[i].to)] = 1;
    }
    int k = 0;
    while(v[k]) k++;
    return sg[u] = k;
}

int main() {
    int x,m,k,pos,ans;
    while(scanf("%d", &n)!=EOF){
        cnt = 0;
        for(int i  = 0; i <=n ; i++) head[i]=-1, sg[i] = -1;
        for(int i = 0; i < n; i++){
            scanf("%d", &x);
            while(x--){
                scanf("%d",&k);
                add(i, k);
            }
        }
        while(~scanf("%d", &m) && m){
            ans = 0, pos = 0;
            for(int i = 0; i < m; i++) {
                scanf("%d", &pos);
                ans ^= getsg(pos);
            }
			if(ans) printf("WIN\n");
			else printf("LOSE\n");
        }
    }
}

~~~

#### 4.    0/1分数规划

•0/1分数规划模型是给定整数a_1,…,a_n 和b_1,…,b_n，求一组解x_i(1≤i≤n, x_i  = 0 || x_i  = 1)，使下列式子最大或最小

•    
$$
\frac {\sum_{i=1}^na_i∗x_i}{\sum_{i=1}^nbi∗x_i}
$$
•方法是二分和dinkelbach算法。

**POJ 2976**

~~~C++
#include <iostream>
#include <cstring>
#include <cmath>
#include <string>
#include <vector>
#include <algorithm>
#include <cstdio>
#include <queue>
#include <set>
using namespace std;
const int maxn = 1e5 + 10;
const int INF = 2147483647;
int n, t, m, k;
int a[maxn],b[maxn];
double res[maxn],ans;
bool check(double x)
{
	for(int i  = 0; i < n; i++)	res[i] = a[i] - x * b[i];
	sort(res, res+n);
	ans = 0;
	for(int i = k ; i < n; i++)	ans += res[i];
	return ans >=0;
}

int main()
{
	while(cin >> n >> k&&n){
		for(int i = 0; i < n; i++)	cin >> a[i];
		for(int i = 0; i < n; i++)	cin >> b[i];
		double l = 0, r = INF;
		while(abs(r-l) >= 1e-6){
			double mid = (l + r) / 2;
			if(check(mid))	l = mid;
			else r  = mid;
		}
		ans = int (l *100+0.5);
		cout << ans <<endl;
	}
	//system("pause");
	return 0;
}
~~~



### 5. 数据结构

####    1.  单调栈

**HDU 1506**

~~~C++
#include <iostream>
#include <queue>
#include <stack>
#include <string>
#include <cstring>
#include <algorithm>
#include <cstdio>
#include <cmath>
#include <set>
using namespace std;
typedef long long ll;
const int N = 1e5+10;
int n,m,p;
ll ans;
int a[N];
int s[N],w[N];

int main()
{
        while(cin >> n&&n){
                ans = 0;
                //memset(s ,0,sizeof(s));
                for(int i = 1; i <= n; i++)      cin>>a[i];
                a[n+1] = p = 0;   
                for(int i = 1; i <= n + 1; i++){
                        if(a[i] > s[p]){
                                s[++p] = a[i]; w[p] = 1;
                        }
                        else{
                                ll width = 0;
                                while(a[i] < s[p]){
                                        width +=w[p];
                                        ans =max(ans, (ll)width * s[p]);
                                        p--;
                                }
                                s[++p] = a[i] ; w[p] = width+1;
                        }
                }
                cout << ans  <<  endl;
        }
        
}       

~~~

#### 2.单调队列

##### 滑动窗口

本题大意是给出一个长度为 n 的数组，编程输出每 k 个连续的数中的最大值和最小值。

~~~C++
#include <iostream>
#include <stack>
#include <string>
#include <cstring>
#include <algorithm>
#include <cstdio>
#include <vector>
#include <set>
using namespace std;
const int N = 1e6+10;
const int MOD = 1e9;
int a[N],q[N];
int t,k,n,tot;

int main()
{
        ios::sync_with_stdio(false);
        cin>>n>>k;
        for(int i = 0; i < n; i++) cin>>a[i];
        int hh = 0,tt = -1;
        for(int i = 0; i < n; i++){
                if(hh <= tt && q[hh] < i-k+1) hh++;
                while(hh <= tt && a[q[tt]] >= a[i]) tt--;
                q[++tt] = i;
                //cout<<i<<" "<<k-1<<endl;
                if(i >= k-1)    cout<<a[q[hh]]<<" ";
        }
        cout<<endl;
        hh = 0,tt = -1;
        for(int i = 0; i < n; i++){
                if(hh <= tt && q[hh] < i-k+1) hh++;
                while(hh <= tt && a[q[tt]] <= a[i]) tt--;
                q[++tt] = i;
                //cout<<i<<" "<<k-1<<endl;
                if(i >= k-1)    cout<<a[q[hh]]<<" ";
        }
        cout<<endl;
}
~~~





####    2. 笛卡尔树

- **作用**：范围最值查询，范围top k查询
- 数列构造笛卡尔树线性时间完成，采用基于栈的算法
- **中序遍历可输出原数列**
- 任意树节点对应数值大/小于其左、右子树内任意接待节点对应数值

##### **范围最值查询与最低公共祖先**（lca)

- 将定义在数列上RMQ问题转化为定义在树结构最低公共祖先问题

##### 建树（节点数值小于u左右子树）

构造笛卡尔树的过程：

使用数据结构栈，栈中保存的始终是右链，即根结点、根结点的右儿子、根结点的右儿子的右儿子……组成的链
并且栈中从栈顶到栈底key依次减小

如果按照从后到前的顺序判断一个元素是否大于A[i]，则每次插入的时间复杂度为O(k+1)
k为本次插入中移除的右链元素个数。因为每个元素最多进出右链各一次，所以整个过程的时间复杂度为O(N)。

~~~c++
struct node
{
    int key;
    int par;
    int l;
    int r;
}tree[maxn];


void init()
{
    for(int i=0;i<maxn;i++){
        tree[i].par=tree[i].l==tree[i].r=-1;//初始化
    }
}

int Build()//建笛卡尔树
{
    int k,top=-1;
    int stack[maxn];
    for(int i=0;i<maxn;i++){
            k=top;
            while(k>=0&&a[stack[k]]>a[i])//栈中比当前元素大的都出栈
            k--;

            if(k != -1){//find it，栈中元素没有完全出栈，当前元素为栈顶元素的右孩子
                tree[i].par = stack[k];
                tree[stack[k]].r = i;
            }
    
            if(k != top){   //出栈的元素为当前元素的左孩子
                tree[stack[k+1]].par = i;
                tree[i].l = stack[k+1];
            }
    
            stack[++k]=i;//当前元素入栈
            top=k;
    }
    tree[stack[0]].parent=-1;//遍历完成后的栈顶元素就是根
    return stack[0];
}
int main()
{
    int i;
    Init();
    for(i=0;i<maxnum;i++)
    {
        cin>>a[i];
        tree[i].key=a[i];
    }

    int root=Build_Tree();
    
    return 0;
}
~~~

![480px-Cartesian_tree.svg](C:\Users\Habsburg\Desktop\480px-Cartesian_tree.svg.png)



####    3.  线段树

##### 建树

~~~C++
void build(int s, int t, int p) {
  // 对 [s,t] 区间建立线段树,当前根的编号为 p
  if (s == t) {
    d[p] = a[s];
    return;
  }
  int m = (s + t) / 2;
  build(s, m, p * 2), build(m + 1, t, p * 2 + 1);
  // 递归对左右区间建树
  d[p] = d[p * 2] + d[(p * 2) + 1];
}
~~~

##### 区间查询

区间查询，比如求区间  的总和（即  ）、求区间最大值/最小值等操作。

~~~C++
int getsum(int l, int r, int s, int t, int p) {
  // [l,r] 为查询区间,[s,t] 为当前节点包含的区间,p 为当前节点的编号
  if (l <= s && t <= r)
    return d[p];  // 当前区间为询问区间的子集时直接返回当前区间的和
  int m = (s + t) / 2, sum = 0;
  if (l <= m) sum += getsum(l, r, s, m, p * 2);
  // 如果左儿子代表的区间 [l,m] 与询问区间有交集,则递归查询左儿子
  if (r > m) sum += getsum(l, r, m + 1, t, p * 2 + 1);
  // 如果右儿子代表的区间 [m+1,r] 与询问区间有交集,则递归查询右儿子
  return sum;
}
~~~

##### 区间修改与懒惰标记

区间修改（区间加上某个值）：

```c++
void update(int l, int r, int c, int s, int t, int p) {
  // [l,r] 为修改区间,c 为被修改的元素的变化量,[s,t] 为当前节点包含的区间,p
  // 为当前节点的编号
  if (l <= s && t <= r) {
    d[p] += (t - s + 1) * c, b[p] += c;
    return;
  }  // 当前区间为修改区间的子集时直接修改当前节点的值,然后打标记,结束修改
  int m = (s + t) / 2;
  if (b[p] && s != t) {
    // 如果当前节点的懒标记非空,则更新当前节点两个子节点的值和懒标记值
    d[p * 2] += b[p] * (m - s + 1), d[p * 2 + 1] += b[p] * (t - m);
    b[p * 2] += b[p], b[p * 2 + 1] += b[p];  // 将标记下传给子节点
    b[p] = 0;                                // 清空当前节点的标记
  }
  if (l <= m) update(l, r, c, s, m, p * 2);
  if (r > m) update(l, r, c, m + 1, t, p * 2 + 1);
  d[p] = d[p * 2] + d[p * 2 + 1];
}
```

区间查询（区间求和）：

~~~C++
int getsum(int l, int r, int s, int t, int p) {
  // [l,r] 为修改区间,c 为被修改的元素的变化量,[s,t] 为当前节点包含的区间,p
  // 为当前节点的编号
  if (l <= s && t <= r) return d[p];
  // 当前区间为询问区间的子集时直接返回当前区间的和
  int m = (s + t) / 2;
  if (b[p]) {
    // 如果当前节点的懒标记非空,则更新当前节点两个子节点的值和懒标记值
    d[p * 2] += b[p] * (m - s + 1), d[p * 2 + 1] += b[p] * (t - m),
        b[p * 2] += b[p], b[p * 2 + 1] += b[p];  // 将标记下传给子节点
    b[p] = 0;                                    // 清空当前节点的标记
  }
  int sum = 0;
  if (l <= m) sum = getsum(l, r, s, m, p * 2);
  if (r > m) sum += getsum(l, r, m + 1, t, p * 2 + 1);
  return sum;
}
~~~



### 6. 图论

####    1．树的重心（输入用scanf)

​		树的重心可以通过简单的两次搜索求出

~~~c++
  void dfs(int u, int pa) {    //dfs找重心
    son[u] = 1;
    int res = 0;
    for (int i = head[u]; ~i; i = edge[i].next) {
      int v = edge[i].to;
      if (v == pa) continue;
      dfs(v, u);
      son[u] += son[v];
      res = max(res, son[v]);
    }
    res = max(res, n - son[u]);
    if (res < siz) {
      ans = u;
      siz = res;
    }
  }
  int getCenter(int x) {
    ans = 0;
    siz = INF;
    dfs(x, -1);
    return ans;
  }
~~~
####    2．树的直径

注：•当树中有负权边不能用dfs来求直径！只能采用树形dp！

•树形dp: 复杂度O(n)，能处理负权边，不能找出两端位置

•两次dfs: 复杂度O(2n)，不能处理负权边，能找出两端位置

1. 两次dfs，复杂度为𝑂(𝑛)

~~~C++
void dfs(int u,int t)
{
    for(int i=head[u]; i!=-1; i=edge[i].next)
    {
        int v=edge[i].v;
        if(vit[v]==0)
        {
            vit[v]=1;
            d[v]=t+edge[i].l;
            if(d[v]>ans)
            {
                ans=d[v];
                node=v;
            }
            dfs(v,d[v]);
        }
    }
}

    memset(vit,0,sizeof(vit));  //加在主函数里
    vit[1]=1;
    ans=0;
    dfs(1,0);
    
    memset(vit,0,sizeof(vit));
    vit[node]=1;
    ans=0;
    dfs(node,0);
~~~



  		2. 树形dp，复杂度为O(n)

~~~c++
void dp(int u)
{
	vis[u] = 1;
	for (int i = head[u]; ~i; i = edge[i].next)
	{
		int v = edge[i].to;
		if(vis[v])	continue;
		dp(v);
		ans = max(ans, d[u] + d[v] + edge[i].w);
		d[u] = max(d[u], d[v] + edge[i].w);
	}
}

		dp(k);//主函数里
		ans=2*sum-ans;//sum为所有边权的和
~~~





####    3.拓扑排序

~~~c++
void work()
{
    memset(vis, 0, sizeof(vis));
    for (int i = n; i >0; i--)
    {
        int q =-1;
        for (int j = n -1; j >=0; j--)
            if (!vis[j] &&in[j] ==0)
            {
                q = j;
                break;
            }
        if (q ==-1)
        {
            printf("-1\n");
            return;
        }
        vis[q] =true;
        ans[q] = i;
        for (int j =0; j < n; j++)
            if (g[q][j])
                in[j]--;
    }
    printf("%d", ans[0]);
    for (int i =1; i < n; i++)
        printf(" %d", ans[i]);
    putchar('\n');
}


        if (!g[b][a])//在输入中注意记录入度
            in[a]++;
        g[b][a] =true;

~~~



#### 4.dijkstra

•在稠密图：朴素版dijkstra  ***O(n2)***

•在稀疏图：堆优化版dijkstra  ***O(mlogn)***

1.朴素算法

~~~C++
void Dijkstra(int src){
    for(int i=1; i<=n; ++i)
        d[i] = INF;
    d[src] = 0; 
    memset(vis, 0, sizeof(vis));
    for(int i=1; i<=n; ++i){
        int u=-1;
        for(int j=1; j<=n; ++j)if(!vis[j]){
            if(u==-1 || d[j]<d[u]) u=j;
        }
        vis[u] = 1;
        for(int j=1; j<=n; ++j)if(!vis[j]){
            int tmp = d[u] + w[u][j];
            if(tmp<d[j]) d[j] = tmp;
        }
    }
}
~~~

2.堆优化

~~~c++
void dijkstra() {
priority_queue<PII, vector<PII>, greater<PII>> q;
    q.push({0, 1});//first存距离,second存节点编号
    while(q.size())   {
        int x = q.top().second; q.pop();
        if(v[x] == 1) continue; v[x] = 1;
        for(int i = head[x]; i; i = nex[i]){ //遍历每一条边O(m)
            int y = ver[i], z = edge[i];
            if(dist[y] > dist[x] + z) {
                dist[y] = dist[x] + z; q.push({dist[y], y});
            }
        }
    }
}
~~~



#### 5.SPFA（Shortest Path Faster Algorithm）

**三角不等式**: 在一个图中的一条边(x, y, z), 有d[y]<= d[x] + z,
如果每一条边都满足三角不等式, 那么d数组就是最短路。

很多时候我们并不需要那么多无用的 relax 操作。

很显然，只有上一次被  relax的结点，所连接的边，才有可能引起下一次的relax  。

那么我们用队列来维护“哪些结点可能会引起relax  ”，就能只访问必要的边了。

**spfa是bellman-ford优化**

##### 1.求最短路（含负权边）

~~~C++
void spfa(int u)
{
	memset(d, 0x3f, sizeof(d)), memset(vis, 0, sizeof(vis));
	d[u] = 0,vis[u] = 1, q.push(u);
	while(!q.empty()){
		int x = q.front();q.pop();
		vis[x] = 0;
		for(int i = head[u]; ~i; i = edge[i].next){
			int y = edge[i].to, z =edge[i].w;
			if(d[y] > d[x] + z){
				d[y] = d[x] + z;
				if(!vis[y])	q.push(y), vis[y] = 1;
			}
		}
	}
}

~~~

##### 2.判断负环

|      **算法名称**       | **能否处理负权边** | **复杂度**  |
| :---------------------: | :----------------: | :---------: |
| dijkstra/堆优化dijkstra |        不能        |   O(n^2)    |
|     堆优化dijkstra      |        不能        |  O(mlogn)   |
|      bellman-ford       |         能         |    O(nm)    |
|          spfa           |         能         | O(km)~O(nm) |

设res(x)表示从1~x的最短路径包含的边数，若发现res(x)≥n，则图中有负环。

~~~c++
bool spfa()
{
	memset(d, 0x3f, sizeof(d)),memset(vis,0,sizeof(vis));
	d[1] = 0,vis[1] = 1,  res[1] = 0;
        q.push(1);
	while(!q.empty()){
		int x = q.front();q.pop();
		vis[x] = 0;
		for(int i = head[x]; ~i; i = edge[i].next){
			int y = edge[i].to, z =edge[i].w;
			if(d[y] > d[x] + z){
				d[y] = d[x] + z;
				res[y] = res[x] + 1;
				if(res[y] >= n)	return true;				
				if(!vis[y]){
						q.push(y), vis[y] = 1;
				}

			}
		}
	}
	return false;
}
~~~

##### 3.差分约束

**差分约束系统**(system of different constraints)是一种特殊的N元一次不等式组，给出n个变量和m个约数条件，其中每个约束条件形如xi-xj≤ck，我们要解决的问题就是求一组解: x_i =a_i使得所有约束条件得到满足。

差分约束系统中的每个约束条件  都可以变形成  ，这与单源最短路中的三角形不等式  非常相似。因此，我们可以把每个变量  看做图中的一个结点，对于每个约束条件  ，从结点  向结点  连一条长度为  的有向边。

~~~C++
bool spfa()
{
	for(int i = 0; i <= n; i++){
		d[i] = INF;
		vis[i] = 0;
	}
	d[1] = 0,vis[1] = 1,res[1] = 0;
        q.push(1);
	while(!q.empty()){
		int x = q.front();q.pop();
		vis[x] = 0;
		for(int i = head[x]; ~i; i = edge[i].next){
			int y = edge[i].to, z =edge[i].w;	
			if(d[y] > d[x] + z){                 //要求最长的距离应求最短路
				d[y] = d[x] + z;
				res[y] = res[x] + 1;
				if(!vis[y]){
					if(res[y] >= n)	return true;	
					q.push(y), vis[y] = 1;
				}
			}
		}
	}
	return false;
}

	for(int i = 1; i <= m; i++){          //距离不大于
		scanf("%d%d%d", &u, &v, &w);
		add(u,v,w);
	}
	for(int i = 1; i <= k; i++){          //距离不小于
		scanf("%d%d%d", &u, &v, &w);
		add(v,u,-w);
~~~



#### 6.Floyd

##### 1.求关系闭包(传递闭包)

POJ 3660

~~~c++
#include <iostream>
#include <cstring>
#include <cmath>
#include <string>
#include <vector>
#include <algorithm>
#include <cstdio>
#include <queue>
#include <set>
using namespace std;
const int maxn = 1e4 + 10;
const int INF = 2147483647;
int n, t, m;
int ans, sum, d[maxn][maxn];     	//Floyd一般用邻接矩阵存图
int main()
{
	cin>>n>>m;	
	int u, v;
	for(int i = 1; i <= n; i++)	
		for(int j=1; j<= n;j++)
			d[i][j]=INF;
	for(int i = 1; i <= m; i++){
			scanf("%d%d", &u, &v);
			d[u][v] = 1;               //u胜v为1
			d[v][u] = -1;              //v负u为-1
	}
	for(int k = 1;k <= n; k++)
		for(int i = 1; i <= n; i++)
			for(int j=1; j <= n; j++){
				if(d[i][k] == d[k][j] && (d[i][k] == 1 || d[i][k] == -1)) //求关系闭包，若 //若A→B，B→C，则A→C
					d[i][j] = d[i][k];
			}
	for(int i = 1; i<= n; i++){
		sum = 0;
		for(int j = 1; j <= n; j++){
			if(d[i][j]!=INF)	sum++;
		}
		if(sum == n-1) ans++;
	}
	cout << ans << endl;
	//system("pause");
	return 0;
}
~~~

##### 2.多源最短路

POJ 2240（边权值相乘最大值的路径）

注：不一定是三种货币之间的兑换，要维护一个相对优越的值

~~~C++
#include <iostream>
#include <cstring>
#include <cmath>
#include <string>
#include <vector>
#include <algorithm>
#include <cstdio>
#include <queue>
#include <set>
#include <map>
typedef long long ll;
using namespace std;
const int maxn = 1e3 + 10;
const int INF = 2147483647;
int n,  m;
bool flag;
double w, d[maxn][maxn];
char s[maxn],f[maxn],t[maxn];
map<string, int> ma;
int sum;
int main()
{
	int icase = 0;
	while(cin>>n&&n){	
	flag = 0;
	for(int i = 1; i <= n; i++){
		scanf("%s",&s);
		ma[s] = i;
		//cout<<i<<endl;
	}
	cin>>m;
	for(int i = 1; i <= n; i++)	
		for(int j=1; j<= n;j++)
			d[i][j]=1;
	for(int i = 1; i <= m; i++){
		cin>>f>>w>>t;
		d[ma[f]][ma[t]] = w;
	}
	
	 for(int k = 1;k <= n; k++)
		for(int i = 1; i <= n; i++)
			for(int j=1; j <= n; j++){
				if(d[i][k]* d[k][j]*1.0 > d[i][j]){  
					d[i][j] = d[i][k] * d[k][j]*1.0;
				}
			}
	for(int i = 1; i <= n; i++)
		if(d[i][i]>1){
			flag=1;
			break;
		}
	if(flag) printf("Case %d: Yes\n",++icase);
	else printf("Case %d: No\n",++icase);
	}
	//system("pause");
	return 0;
}
~~~



#### 7.LCA(最近公共祖先)

- **定义**：两个节点的最近公共祖先，就是这两个点的公共祖先里面，离根最远的那个。 

- **性质**：1.两点最近公共祖先必在树上两点间最短路上

  ​			2.d是树上两点间距离，h为点到树根的距离
  $$
  d(u,v) = h(u) + h(v) -2h(LCA(u,v))
  $$

**Tarjan算法**

- 使用并查集对“向上标记法”的优化
- 三类节点：1.vis[x] == 2 已访问并回溯
      2.vis[x] == 1 已开始递归但未回溯，正在访问的节点x以及x的祖先
      3.vis[x] == 0 尚未访问的节点

~~~C++
int getf(int x) //寻找父亲
{
	return f[x] == x ? x : f[x] = getf(f[x]);
}

void Tarjan(int u)
{
	vis[u] = 1;
	int to;
	for (int i = head[u]; ~i; i = edge[i].next)
	{
		to = edge[i].to;
		if (vis[to])	continue;
		d[to] = d[u] + edge[i].w;
		Tarjan(to);
		f[to] = u;
	}
	for (int i = 0; i < query[u].size(); i++)
	{
		int to = query[u][i],id = quert_id[u][i];
		if (vis[to] == 2)
		{
			int lca = getf(to);
			ans[id] = min(ans[id], d[u] + d[to] - 2 * d[lca]);
		}
	}
	vis[u] = 2;      //一回溯，标记为2
}
~~~

##### 小技巧：存储离线一次性读入

可以用
$$
vector<int> query[N]
$$

~~~C++
vector<int> query[maxn], quert_id[maxn]; 
void add_query(int x, int y, int id)  //只要有关，就要加入询问
{
	query[x].push_back(y), quert_id[x].push_back(id);
	query[y].push_back(x), quert_id[y].push_back(id);
}		
for (int i = 1; i <= m; ++i)   //读入询问！！！学会方法
{
	int x, y;
	scanf("%d%d", &x, &y);
	if(x == y) ans[i] = 0;
	else {
    add_query(x,y,i);
	ans[i] = 1 << 30;
	}
}
~~~



#### 8.二分图匹配（染色法）

**定理**：一张图是二分图，当且仅当图中不存在奇环(长度为奇数的环)。

**方法**：121212…染色，没有矛盾就是二分图

**CF575 div3 E Cover it!**

~~~C++
void dfs(int u, int c)
{
	color[u] = c;
	for(int i = head[u]; ~i; i = edge[i].next){
		int v = edge[i].to;
		if(!color[v]){
			dfs(v,3-c);
		}
	}
}
int main()
{
	cin >> t;
	while(t--){
		scanf("%d%d", &n,&m);
		for(int i = 1; i <= n; i++)
		color[i]=0,head[i]=-1;        //不要用memset，会超时！！！
		cnt = 0;
		int u, v;
		for(int i = 0; i < m; i++){
			scanf("%d%d",&u,&v);
			add(u,v);
			add(v,u);
		}	
		dfs(1,1);
		k = 0, l = 0;
		for(int i = 1; i <= n; i++){
			if(color[i] == 1)	a[++k] = i;
			else      b[++l] = i;
		}
		bool flag;
		if(k <= n/2)	flag = 1;
		else    flag = 0;
		if(flag){
			printf("%d\n",k);
			for(int i = 1; i <=k; i++)	printf("%d ",a[i]);
			cout<<endl;
		}
		else
		{
			printf("%d\n",l);
			for(int i = 1; i <=l; i++)	printf("%d ",b[i]);
			cout<<endl;
		}
	}
	//system("pause");
	return 0;
}
~~~

也可用bfs

~~~c++
bool bfs1(int s)
{
	queue<int> q;
	color[s] = 1;
	q.push(s);
	while(q.size()){
		int u = q.front();
		q.pop();  
		for(int i = head[u]; ~i; i = edge[i].next){
			int v = edge[i].to;
			if(!color[v]){
				color[v] = 3-color[u];
				q.push(v);	
			}
			else if(color[v] == color[u] )	return false;
		}
	}
	return true;
}
~~~



#### 9.二分图最大匹配（匈牙利算法）

**最大匹配**：“任意两条边都没有公共端点”的边的集合被称为图的一组**匹配**，在二分图中，包含边最多的一组匹配被称为二分图的最大匹配。5

**寻找增广路**：1.y本身是非匹配点

​						2.y已与左部点x1匹配，但从x1出发可找到另一个右部点y1匹配，此时x~y~x1~y1是一条增广路

**核心代码**：

~~~C++
bool dfs2(int u)
{
	for(int i = head[u]; ~i; i = edge[i].next){
		int v = edge[i].to;
		if(!vis[v]){
			vis[v] = 1;
			if(!match[v] || dfs2(match[v])){
				match[v] = u;
				return true;
			}
		}
	}
	return false;
}
for(int i = 1; i <= n; i++){
	for(int j = 1; j <= n; j++) vis[j] = 0;
		if(dfs2(i))    ans++;
}
~~~



#### 10.Tarjan求割边

**时间戳**: 在图的深度优先遍历过程中，按照每个节点第一次被访问的时间顺序，依次给予1~n的数字标记，该标记被称为”时间戳”。***dfn[i]***表示

***subtree(x)***表示搜索树中以x为根的子树结点的集合。

***low(x)*****（追溯值）**表示顶点x及其子树中的点，通过非搜索树上的边，能够回溯到的最早的点(dfn最小)的dfn值(但不能连接x与其父节点的边)。

**割边判定法则**：
$$
dfn(x)<low(y)
$$
**核心代码**

~~~c++
void tarjan1(int u, int in_edge)
{
	dfn[u] = low[u] = ++num;
	for(int i = head[u]; ~i; i = edge[i].next){
		int v = edge[i].to;
		if(!dfn[v]){
			tarjan1(v, i);
			low[u] = min(low[v], low[u]);
			if(low[v] > dfn[u]){
				bridge[i] = bridge[i ^1] = true;
			}
		}
		else if(i != (in_edge ^ 1) )
			low[u] = min (low[u],dfn[v]);
	}
}
~~~



#### 11.Tarjan求割点

**割点判定法则**：
$$
𝑑𝑓𝑛(𝑥)≤𝑙𝑜𝑤(𝑦)
$$
**核心代码**

~~~c++
	void tarjan2(int u)
	{
		dfn[u] = low[u] = ++num;
		int flag = 0;
		for(int i = head[u]; ~i; i = edge[i].next){
			int v = edge[i].to;
			if(!dfn[v]){
				tarjan2(v);
				low[u] = min(low[v], low[u]);
				if(low[v] >= dfn[u]){
					flag++;
					if(flag > 1 || u != root)cut[u] = true;
				}
			}
			else  low[u] = min (low[u],dfn[v]);
		}
	}

//加边中
if(u == v) continue;
//主函数中
for(int i = 1;i <= n; i++){
    if(!dfn[i]) root = i, tarjan2(i);
}
for(int i = 1;i <= n; i++){
	if(cut[i])	printf("%d",i);
}
~~~

#### 12.网络流

**最大流**

整个网络流量最大，流函数f为最大流

1. 增广路算法

   - 各条边剩余流量大于0，为一条增广路，不断用/bfs求增广路

   - O(nm^2),1000~10000
   - 为了存反向边，邻接表成对存储
   - （x,y）减小e, (y,x)增大e

~~~c++
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;
const int N = 1010, M = 10010, inf = 1 << 29;
int head[N], next[N], ver[M], edge[M], v[N], incf[N], pre[N];
int n, m, s, t, tot, maxflow;
void add(int x, int y, int z){
    ver[++tot] = y; edge[tot] = z; next[tot] = head[x]; head[x] = tot;
    ver[++tot] = x; edge[tot] = 0; next[tot] = head[y]; head[y] = tot;
}

bool bfs(){
    memset(v,0,sizeof(v));
    queue<int> q;
    q.push(s), v[s] = 1;
    incf[s] = inf;    //增广路各边最小剩余流量
    while(q.size()){
        int x = q.front();
        q.pop();
        for(int i = head[x]; i; i = next[i]){
            if(edge[i]){
                int y = ver[i];
                if(v[y]) continue;
                incf[y] = min(incf[x], edge[i]);  //最小剩余容量作为此条增广路流量
                pre[y] = i;
                q.push(y), v[y] = 1;
                if(y == t) return 1;
            }
        }
    }
    return 0;
}

void update(){//更新增广路和反向边剩余流量
    int x = t;
    while(x != s){
        int i = pre[x];
        edge[i] -= incf[t];
        edge[i^1] += incf[t]; //“成对存储"xor 1 ,找到反向的容量        
        x = ver[i^1];
    }
    maxflow += incf[t];
}

int main(){
    while(cin >> n >> m){
        memset(head, 0, sizeof(head));
        s = 1; t = n; tot = 1; maxflow = 0;
        for(int i = 0; i < m; i++){
            int x, y, z;
            scanf("%d%d%d", &x, &y, &z);
            add(x, y ,z);
        }
        while(bfs()) update();
        cout << maxflow << endl; 
    }
}
/*
7 10
1 2 4
1 3 5
2 3 1
2 4 5
3 4 2
3 5 5
4 7 5
5 4 6
5 6 3
6 7 2
*/

~~~

2. **Dinic算法**（更快，1e4~1e5）
1. BFS求出节点层次，构造分层图
   2. 分层图DFS找增广路，回溯时更新剩余容量，每个点可以流向多条边，还要剪枝

~~~C++
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;
const int N = 5010, M = 30010, inf = 1 << 29;
int n,m,s,t,tot,maxflow;
int ver[M], next[M], head[N], edge[M], d[N];
queue<int> q;
void add(int x, int y, int z){
    ver[++tot] = y; edge[tot] = z; next[tot] = head[x]; head[x] = tot;
    ver[++tot] = x; edge[tot] = 0; next[tot] = head[y]; head[y] = tot;
}

bool bfs(){
    memset(d,0,sizeof(d));
    while(q.size()) q.pop();   //每一次生成分层图之前先清空栈
    q.push(s), d[s] = 1;
    while(q.size()){
        int x = q.front(); q.pop();
        for(int i = head[x]; i; i = next[i]){
            if(edge[i] && !d[ver[i]]){   //d[ver[i]] (与x相连的下一个点）不为零，说明已构成了分层图中的一层，不满足条件
                d[ver[i]] = d[x] + 1;
                q.push(ver[i]);
                    //cout << "1" <<endl;
                if(ver[i] == t) return 1;//可以到达汇点，说明可以构成一个分层图
            }
        }
    }
    return 0;
}

int dinic(int x, int flow){
    if(x == t) return flow;
    int rest = flow, k;
    for(int i = head[x]; i && rest; i = next[i])   //rest为什么不能等于0
        if(edge[i] && d[ver[i]] == d[x] + 1){
            k = dinic(ver[i], min(edge[i], rest));
            if(!k) d[ver[i]] = 0;   //剪枝，去掉增广完毕的点(什么意思？)
            edge[i] -= k;
            edge[i^1] += k;
            rest -= k; 
        }
    return flow - rest;   //相当于返回 k（已经找到的流流量）
}

int main(){
    while(cin >> n >> m){
        s = 1; t = n; tot = 1; maxflow = 0;
        for(int i = 0; i < m; i++){
            int x, y, z;
            scanf("%d%d%d", &x, &y, &z);
            add(x, y ,z);
        }
        int flow = 0;
        while(bfs())
            while(flow = dinic(s, inf)) cout << flow <<endl, maxflow += flow;
        cout << maxflow <<endl;
    }
    system("pause");
}

~~~

   

 

 