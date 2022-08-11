### 牛的旅行

![image-20220809191826180](http://www.cdn.liver0377.xyz/typora/202208091918244.png)

![image-20220809191842310](http://www.cdn.liver0377.xyz/typora/202208091918354.png)





**解题思路**

![image-20220809191954815](http://www.cdn.liver0377.xyz/typora/202208091919862.png)

**代码实现**

```cc
#include <iostream>
#include <cstring>
#include <cmath>
using namespace std;

const int N=160,INF=0x3f3f3f3f;
typedef pair<int,int> PII;

double d[N][N];
double maxd[N];
int n;
char g[N][N];
PII q[N];

double get_distance(int i,int j){
    int x1=q[i].first,y1=q[i].second;
    int x2=q[j].first,y2=q[j].second;
    return sqrt(pow((x1-x2),2)+pow((y1-y2),2));
}

void floyd(){
    for(int k=1;k<=n;k++){
        for(int i=1;i<=n;i++){
            for(int j=1;j<=n;j++){
                d[i][j]=min(d[i][j],d[i][k]+d[k][j]);
            }
        }
    }
}

int main(){
    scanf("%d",&n);
    //读入坐标
    for(int i=1;i<=n;i++){
        scanf("%d%d",&q[i].first,&q[i].second);
    }
    //读入邻接矩阵
    for(int i=1;i<=n;i++){
        for(int j=1;j<=n;j++){
            cin>>g[i][j];
        }
    }
    //建图
    for(int i=1;i<=n;i++){
        for(int j=1;j<=n;j++){
            if(i==j) d[i][j]=0;
            else{
                if(g[i][j]=='1') d[i][j]=get_distance(i,j);
                else d[i][j]=INF;
            }
        }
    }

    floyd();
    //每个连通块中两点的最长距离
    double max1=-INF;
    for(int i=1;i<=n;i++){
        for(int j=1;j<=n;j++){
            if(d[i][j]<INF/2){  //如果i点和j点在一个连通块
                maxd[i]=max(maxd[i],d[i][j]);
            }
        max1=max(max1,maxd[i]);
        }
    }
    //两个连通块连一条边的最大距离的最短距离
    double max2=INF;
    for(int i=1;i<=n;i++){
        for(int j=1;j<=n;j++){
            if(d[i][j]==INF)   //如果当前连通块不连通,才可以连接
                max2=min(max2,maxd[i]+maxd[j]+get_distance(i,j));
        }
    }

    printf("%.6lf\n",max(max1,max2));
    return 0;

}

```







### 排序

![image-20220811152020094](http://www.cdn.liver0377.xyz/typora/202208111520161.png)

![image-20220811152034300](http://www.cdn.liver0377.xyz/typora/202208111520343.png)



**解题思路**

- 设`g`保存原图, `d`保存传递闭包, 若`i < j`, 则`d[i][j] = 1`

- 每读取一个关系时，就求当前状态的传递闭包， 然后判断一下当前的状态

  - 0: 还没有判断完毕

    此时对于任意点`i`来说，一定存在着一个点`j(j != i)`,使得`d[i][j] == 0 && d[j][i] == 0`

  - 1: 发生矛盾

    发生矛盾时，一定存在点`i`,`j`, 使得`g[i][j] == 1` && `g[j][i] == 1`

    此时求传递闭包，一定会导致`d[i][i] = d[j][j] = 1`

  - 2: 判断完毕

    只要将上面两种情况排除，接下来就是判断完毕的情况了

- 如何确定顺序

  每次从未访问的点当中，选出一个小于所有其它未访问的点的点，将其标记为已访问，然后取出

**代码实现**

```cc
#include <iostream>
#include <algorithm>
#include <cstring>

using namespace std;

const int N = 26;

bool g[N][N];   // 存储原图
bool d[N][N];   // 存储闭包
bool st[N];     // 每个点是否已经判断成功，用于顺序输出
int n, m;

void floyd() {
    memcpy(d, g, sizeof d);
    
    for (int k = 0; k < n; k++) {
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < n; j++) {
                d[i][j] |= d[i][k] & d[k][j];
            }
        }
    }
}

int check() {
    // 1. 是否矛盾
    for (int i = 0; i < n; i++) {
        if (d[i][i]) return 1;
    }
    
    // 2. 是否判断完毕
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < i; j++) {
            if (!d[i][j] && !d[j][i]) {
                return 0;
            }
        }
    }
    
    return 2;
}

char get_min() {
    for (int i = 0; i < n; i++) {
        if (!st[i]) {
            bool flag = true;
            for (int j = 0; j < n; j++) {
                if (!st[j] && d[j][i]) {
                    flag = false;
                    break;
                }
            }
            if (flag) {
                st[i] = true;
                return i + 'A';
            }
        }
    }
    return ' ';
}

int main() {
    while (cin >> n >> m, n || m) {
        memset(g, 0, sizeof g);
        int type = 0;      // 0: 还没有判断完毕
                           // 1: 发生矛盾
                           // 2: 判断完毕
        int t;             
        for (int i = 0; i < m; i++) {
            char str[5];
            cin >> str;
            int a = str[0] - 'A', b = str[2] - 'A';
            if (type == 0) {
                g[a][b] = 1;
                floyd();    // 求出传递闭包
                type = check();
                t = i;
            }    // 当type == 1或者2时不能直接break, 因为有多组测试数据，否则后面的例子会读取到错误数据
        }
        
        t += 1;
       
        if (type == 0) {
            // cout << "t: " << t << endl;
            cout << "Sorted sequence cannot be determined." << endl;
        } else if (type == 1) {
            cout << "Inconsistency found after " << t << " relations." << endl;
        } else {
            cout << "Sorted sequence determined after " << t << " relations: ";
            memset(st, 0, sizeof st);
            for (int i = 0; i < n; i++) {
               cout << get_min();
            }
            cout << ".\n";
        }
    }
    return 0;
}
```







###  观光之旅

![image-20220811162736456](http://www.cdn.liver0377.xyz/typora/202208111627516.png)



**解题思路**

- 如果从集合的角度来考虑所有的环，设环中的最大节点编号为`k`, 那么可以根据`k`进行不重不漏的分组

  ![image-20220811163007223](http://www.cdn.liver0377.xyz/typora/202208111630272.png)

- 再考虑一下`Floyd`的定义

  ![image-20220811163050360](http://www.cdn.liver0377.xyz/typora/202208111630413.png)

  - 当外层循环`k`刚刚开始时，$d[i][j]$表示的含义依旧是从点`i`到点`j`，经过的节点编号不超过`k - 1`的路径当中，最小的路径长度

  - 此时只需要枚举`i`, `j`,就可以构造出一个环**(i, j, k)**, 其中`i`， `j`均与`k`直接相连

  - 此时`k`就是该环中的最大节点编号，只要枚举所有的`i`,`j`,就能够算出上述集合中的分区`k`, 也就是以`k`为最大节点编号的环中

    长度最小的环, 使用一个全局最小值来记录所有分区的最小值即可

  - 从代码上来看，只需要在`Floyd`的最外层加上一个两层循环即可

- 如何记录最短路路径

  - 使用一个数组`pos`, 在`Floyd`的最内层循环中，使用`pos[i][j]`记录下选取的中间节点
  - 这样，最终就能够确保从`i`走到`j`, 一定能够经过`k`

  - 然后，递归判断(i, k)以及(k, j)即可



**代码实现**

```cc
#include <iostream>
#include <algorithm>
#include <cstring>

using namespace std;

const int N = 110;
const int M = 1e4 + 10;

int d[N][N], w[N][N];
int pos[N][N];
int path[N], cnt;
int n, m;

// 存储从i + 1 ~ j - 1的最小路径
void get_path(int i, int j) {
    // 当从i到j的最短路径就是(i, j)这条边时， pos[i][j] == 0
    if (pos[i][j] == 0) return;
    
    int k = pos[i][j];
    get_path(i, k);
    path[cnt ++] = k;
    get_path(k, j);
}

int main() {
    cin >> n >> m;
    
    memset(w, 0x3f, sizeof w);
    for (int i = 0; i < m; i++) {
        int a, b, c;
        cin >> a >> b >> c;
        w[a][b] = w[b][a] = min(w[a][b], c);      // 防止有重边
    }
    
    for (int i = 1; i <= n; i++) d[i][i] = 0;
    
    memcpy(d, w, sizeof d);
    long long res = 0x3f3f3f3f;
    for (int k = 1; k <= n; k++) {
        // 此时d[i][j]中记录的是经过编号为1 ~ k - 1的从i到j的最短距离, 这里直接令i < j
        for (int i = 1; i < k; i++) {
            for (int j = i + 1; j < k; j++) {
                if ((long long)d[i][j] + w[j][k] + w[k][i] < res) {
                    res = d[i][j] + w[j][k] + w[k][i];
                    cnt = 0;
                    path[cnt ++] = k;
                    path[cnt ++] = i;
                    get_path(i, j);
                    path[cnt ++] = j;
                    
                }
            }
        }
        for (int i = 1; i <= n; i++) {
            for (int j = 1; j <= n; j++) {
                if (d[i][j] > d[i][k] + d[k][j]) {
                    d[i][j] = d[i][k] + d[k][j];
                    pos[i][j] = k;
                }
            }
        }
    }
    
    if (res == 0x3f3f3f3f) puts("No solution.");
    else {
        for (int i = 0; i < cnt; i++) {
            cout << path[i] << " ";
        }
        cout << endl;
    }
    return 0;
}
```







### 牛站

**题干**

![image-20220811184415708](http://www.cdn.liver0377.xyz/typora/202208111844768.png)

**解题思路**

![image-20220811184652251](http://www.cdn.liver0377.xyz/typora/202208111846302.png)

![image-20220811184705123](http://www.cdn.liver0377.xyz/typora/202208111847221.png)

**代码实现**

```cc
#include<iostream>
#include<cstring>
#include<map>

using namespace std;

const int N=210;

int res[N][N],g[N][N];
int k,n,m,S,E;
map<int,int> id;

// 计算a * b, c中存储结果
void mul(int c[][N],int a[][N],int b[][N])
{
    static int temp[N][N];
    memset(temp,0x3f,sizeof temp);
    for(int k=1;k<=n;k++)
        for(int i=1;i<=n;i++)
            for(int j=1;j<=n;j++)
                temp[i][j]=min(temp[i][j],a[i][k]+b[k][j]);
    memcpy(c,temp,sizeof temp);
}

void qmi()
{
    memset(res,0x3f,sizeof res);
    for(int i=1;i<=n;i++) res[i][i]=0;//经过0条边
    while(k)//更新的过程
    {
        if(k&1) mul(res,res,g);//res=res*g;根据k决定是否用当前g的结果去更新res
        mul(g,g,g);//g=g*g;g的更新
        k>>=1;
    }
}

int main()
{
    cin>>k>>m>>S>>E;//虽然点数较多，但由于边数少，所以我们实际用到的点数也很少，可以使用map来离散化来赋予
    //他们唯一的编号
    memset(g,0x3f,sizeof g);
    //这里我们来解释一下为什么不去初始化g[i][i]=0呢？
    //我们都知道在类Floyd算法中有严格的边数限制，如果出现了i->j->i的情况其实在i->i中我们是有2条边的
    //要是我们初始化g[i][i]=0,那样就没边了，影响了类Floyd算法的边数限制！
    if(!id.count(S)) id[S]=++n;
    if(!id.count(E)) id[E]=++n;
    S=id[S],E=id[E];
    while(m--)
    {
        int a,b,c;
        scanf("%d%d%d",&c,&a,&b);
        if(!id.count(a)) id[a]=++n;
        if(!id.count(b)) id[b]=++n;
        a=id[a],b=id[b];
        g[a][b]=g[b][a]=min(g[a][b],c);
    }
    qmi();
    cout<< res[S][E] <<endl;
    return 0;
}
```





