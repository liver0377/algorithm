### tarjan(DCC)

- 割点与桥

  ![image-20220823142832749](http://www.cdn.liver0377.xyz/typora/202208231428863.png)

- 双连通分量

  ![image-20220823142909976](http://www.cdn.liver0377.xyz/typora/202208231429015.png)

  > 这里的“极大”是指加上任何一个点之后，该图都不再是双连通子图

- 性质

  - e-DCC

    - 对于任意一个e-DCC,  删掉连通分量的任意一条边，它都是连通的

    - 任意两个节点之间，至少存在两条没有公共边的路径(充要条件)
    - 环是e-DCC
    - 桥一定是搜索树中的边
    - 给定一张无向图，叶子节点数目为`cnt`, 至少需要$\lceil cnt / 2 \rceil$条边才能够将原图变为边双连通图
    - 一个边双连通图的每一条边，都至少包含在一个简单环中

  - v-DCC

    一张无向图为点双连通图，当且仅当满足下面两个条件之一

    ![image-20220823165106308](http://www.cdn.liver0377.xyz/typora/202208231651345.png)

    



与有向图一样，`tarjan`算法中仍然需要使用到`dfn`以及`low`数组

- `dfn[i]`: 节点`i`在搜索树中的时间戳

- `low[i]`:

  设`subtree(x)`表示搜索树中以`x`为根的子树, `low[x]`定义为以下节点中的最小`dfn`

  ![image-20220823145153711](http://www.cdn.liver0377.xyz/typora/202208231451744.png)

  - 例

    下图中`[]`中的数为`low[x]`

    ![image-20220823145212590](http://www.cdn.liver0377.xyz/typora/202208231452905.png)

**如何判断一个边是桥**

- 设边(x, y)， 其中`y`是`x`在搜索树上的一个子节点，若有

  - $low[y] > dfn[x]$

  则$(x, y)$是桥

  上面这个公式表明从`y`点不能够沿(x, y)之外的边访问到搜索树中`x`前面的点，满足桥的定义





**e-DCC求法**

求解`e-DCC`有两种方法：

1. 将无向连通图中的所有割边全部删除，新图中的每一个连通块就是一个`e-DCC`
2. 使用和求`SCC`一样的方法，使用一个栈来维护当前可能的`e-DCC`

第二种方法更加简单，这里只给出第二种方法的模板

```cc
void tarjan(int u, int from) {
    dfn[u] = low[u] = ++timestamp;
    stk[++ top] = u;
    for (int i = h[u]; ~i; i = ne[i]) {
        int j = e[i];
        if (!dfn[j]) {
            tarjan(j, i);    // i --> (u, j)
            low[u] = min(low[u], low[j]);
            if (low[j] > dfn[u]) is_bridge[i] = is_bridge[i ^ 1] = true;
        } else if (i != (from ^ 1)) {
            low[u] = min(low[u], dfn[j]);
        }
    }
    
    if (low[u] == dfn[u]) {
        int x;
        dcc_cnt ++;
        do {
            x = stk[top --];
            id[x] = dcc_cnt;
        } while (x != u);
    }
}
```

- 注意点

  - `from`表示(fa, u)这条有向边在$e, ne$数组中的下标

  - 无向边是以两条有向边表示的，他们在$e, ne$数组中是以{0, 1}, {2, 3} ... 等一对一对的顺序出现的，因此使用`^`运算符就可以获取到某个有向边对应的另一条有向边
  - `else if`的条件主要是因为当`i`表示的边为(fa, u)的配对有向边时, `low`数组不应该被更新





**如何判断一个点是割点**

- 假设有点`x`
  - 若点`x`不是根节点，且在搜索树中存在一个子节点`y`, 使得`low[y] >= dfn[x]`, 那么`x`为割点
  - 若点`x`是根节点，且在搜索树中存在至少两个子节点`y1, y2`, 使得`low[y1] >= dfn[x], low[y2] >= dfn[x]`, 那么`x`为割点

注意这里的条件$low[y] >= dfn[x]$包含"=", 因此在向上遍历到父节点时，也可以照常更新`low`



**v-DCC求法**

下面是求割点模板

```cc
vodi tarjan(int x) {
    dfn[x] = low[x] = ++timestamp;
    int cnt = 0;
    for (int = h[x]; ~i; i = ne[i]) {
        int j = e[i];
        if (!dfn[j]) {
            tarjan(j);
            low[x] = min(low[x], low[j]);
            if (low[j] >= dfn[x]) {
                cnt ++;
            }
        } else {
            low[x] = min(low[x], dfn[j]);
        }
    }
    if (x != root || cnt > 1) cut[x] = true;  // cut[i]: i是否为割点
}
```



**v-DCC求点连通分量**

```cc
void tarjan(int u) {
    low[u] = dfn[u] = ++timestamp;
    stk[++top] = u;
    
    if(root == u && h[u] == -1) {
        dcc[++dcc_cnt].push_back(u);
        return;
    }
    
    int cnt = 0;
    for (int i = h[u]; ~i; i = ne[i]) {
        int j = e[i];
        if (!dfn[j]) {
            tarjan(j);
            low[u] = min(low[u], low[j]);
            if (low[j] >= dfn[u]) {
                cnt ++;
                dcc_cnt ++;
                if (u != root || cnt >= 2) cut[u] = true;
                int x;
                do {
                    x = stk[top--];
                    dcc[dcc_cnt].push_back(x);
                } while (x != j);
                dcc[dcc_cnt].push_back(u);
            }
        } else {
            low[u] = min(low[u], dfn[j]);
        }
    }
}
```

- 这里需要注意的是弹栈操作移动到了循环内部

  这是因为判断的条件改为了$low[j] >= dfn[u]$, 这导致了当`u`为割点时, $dfn[u]$可能不等于$low[u]$

  

**v-DCC缩点**

![image-20220823195127754](http://www.cdn.liver0377.xyz/typora/202208231951809.png)





### 冗余路径

![image-20220823163228465](http://www.cdn.liver0377.xyz/typora/202208231632534.png)





**解题思路**

- 前面给出的e-DCC的性质表明:

  任意两个节点之间，至少存在两条没有公共边的路径 <=> e-DCC是等价条件

- 题意转换

  给定一张无向连通图，求至少要加多少条边，才能够使得该图变为边双连通图

- 首先使用`tarjan`算法求出e-DCC然后进行缩点, 缩点之后的图一定是一个树

  - 首先因为原图是连通的，因此新图肯定也是一个连通图
  - 其次由于环是e-DCC, 所有的环一定会被缩成点，因此新图肯定没有环

  综上，新图一定是树

- 设`cnt`表示树中的叶子节点数目，则至少应该加$\lceil cnt / 2 \rceil$条边

  - 证明：
    假设树如下图

    ![image-20220823163845980](http://www.cdn.liver0377.xyz/typora/202208231638043.png)

    对于每个叶子节点$a_i$, 可以将其与其它子树的任意叶子节点$a_k$连接一条边，也就是将不同子树的叶子节点之间两两配对

    通过$cnt / 2$次操作之后, 至多只会剩下1个叶子节点，此时将该叶子节点与根节点之间连一条边

    经过上述操作，至少需要$\lceil cnt / 2 \rceil$操作，整张图就会变为边双连通图

    ![image-20220823164423015](http://www.cdn.liver0377.xyz/typora/202208231644056.png) 

- 因此，可以求出缩点之后每个点的入度，入度为1的点就是叶子节点(注意无向边用两条有向边表示，因此内部节点的入度> 1)

**代码实现**

```cc
#include <iostream>
#include <algorithm>
#include <cstring>

using namespace std;

const int N = 5e3 + 10;
const int M = 2e4 + 10;

int h[N], e[M], ne[M], idx;
int dfn[N], low[N], timestamp;
int stk[N], top;
int id[N], dcc_cnt;
int d[N];   // d[i]: 新图中点i的入度
bool is_bridge[M];
int n, m;

void add(int a, int b) {
    e[idx] = b, ne[idx] = h[a], h[a] = idx ++;
}

void tarjan(int u, int from) {
    dfn[u] = low[u] = ++timestamp;
    stk[++ top] = u;
    for (int i = h[u]; ~i; i = ne[i]) {
        int j = e[i];
        if (!dfn[j]) {
            tarjan(j, i);    // i --> (u, j)
            low[u] = min(low[u], low[j]);
            if (low[j] > dfn[u]) is_bridge[i] = is_bridge[i ^ 1] = true;
        } else if (i != (from ^ 1)) {
            low[u] = min(low[u], dfn[j]);
        }
    }
    
    if (low[u] == dfn[u]) {
        int x;
        dcc_cnt ++;
        do {
            x = stk[top --];
            id[x] = dcc_cnt;
        } while (x != u);
    }
}

int main() {
    cin >> n >> m;
    memset(h, -1, sizeof h);
    for (int i = 0; i < m; i++) {
        int a, b;
        cin >> a >> b;
        add(a, b), add(b, a);
    }
    
    // 找出桥并且缩点
    tarjan(1, -1);
    
    for (int i = 0; i < idx; i++) {
        int j = e[i];   // 边的出顶点
        if (is_bridge[i]) d[id[j]] ++;
    }
    
    int res = 0;
    for (int i = 1; i <= dcc_cnt; i++) {
        if (d[i] == 1) res ++;
    }
    cout << (res + 1) / 2 << endl;
    return 0;
}
```









### 电力

![image-20220823180516271](http://www.cdn.liver0377.xyz/typora/202208231805330.png)



**解题思路**

- 首先可以求出原图中连通块的数目`cnt`

- 接下来在每个连通块内部任选一个根节点，在该联通块内部进行深搜，尝试计算出去除每一个割点之后所能够得到的

  新连通块数目，求出全局最大值`ans`， 最终返回`ans + cnt - 1`

- `tarjan`算法本身就具有深搜作用，可以直接用它求连通块数目以及深搜枚举割点

- 注意，对于任意一个连通块，可以随便选取一个点作为根节点，得到的的`ans`肯定是一样的



**代码实现**

```cc
#include <iostream>
#include <cstring>
#include <algorithm>

using namespace std;

const int N = 1e4 + 10;
const int M = 3e4 + 10;

int h[N], e[M], ne[M], idx;
int dfn[N], low[N], timestamp;
int n, m;
int root, cnt, ans;    // root: 求v-DCC时选用的连通块的根节点
                       // cnt: 原图连通块的数目
                       // ans: 对某个v-DCC去除一个割点之后所能够得到的最大连通块数目(不包括其它连通块)
                  
             
void add(int a, int b) {
    e[idx] = b, ne[idx] = h[a], h[a] = idx ++;
}     

void tarjan(int u) {
    low[u] = dfn[u] = ++timestamp;
    
    int cnt = 0;   // u的子树j中满足low[j] >= dfn[u]的个数
    for (int i = h[u]; ~i; i = ne[i]) {
        int j = e[i];
        if (!dfn[j]) {
            tarjan(j);
            low[u] = min(low[u], low[j]);
            if (low[j] >= dfn[u]) cnt ++;
        } else {
            low[u] = min(low[u], dfn[j]); 
        }
    }
    
    if (root == u) ans = max(ans, cnt);   
    else if (root != u) ans = max(ans, cnt + 1);  // 这里u是不是割点都行
}

int main() {
    while (cin >> n >> m, n || m) {
        memset(h, -1, sizeof h);
        memset(dfn, 0, sizeof dfn);
        idx = 0;
        int a, b;
        for (int i = 0; i < m; i++) {
            cin >> a >> b;
            add(a, b); add(b, a);
        }
        
        cnt = 0, ans = 0;
        for (root = 0; root <= n - 1; root++) {
            if (!dfn[root]) {
                cnt ++;
                tarjan(root);
            }
        }
        
        cout << ans + cnt - 1 << endl;
    }
    return 0;
}
```







### 矿场搭建

![image-20220823194901494](http://www.cdn.liver0377.xyz/typora/202208231949572.png)

![image-20220823194911235](http://www.cdn.liver0377.xyz/typora/202208231949274.png)



**解题思路**

![image-20220823194959386](http://www.cdn.liver0377.xyz/typora/202208231949448.png)

> 这里的缩点过程仍然是虚拟的



**代码实现**

```cc
#include <iostream>
#include <algorithm>
#include <cstring>
#include <vector>

using namespace std;

const int N = 510;
const int M = 1010 * 2;

typedef unsigned long long ULL;

int h[N], e[M], ne[M], idx;
int dfn[N], low[N], timestamp;
int stk[N], top;
bool cut[N];
vector<int> dcc[N];   // dcc[i]: 连通分量i的所有节点编号
int dcc_cnt;
int n, m, root;

void add(int a, int b) {
    e[idx] = b, ne[idx] = h[a], h[a] = idx ++;
}

void tarjan(int u) {
    low[u] = dfn[u] = ++timestamp;
    stk[++top] = u;
    
    if(root == u && h[u] == -1) {
        dcc[++dcc_cnt].push_back(u);
        return;
    }
    
    int cnt = 0;
    for (int i = h[u]; ~i; i = ne[i]) {
        int j = e[i];
        if (!dfn[j]) {
            tarjan(j);
            low[u] = min(low[u], low[j]);
            if (low[j] >= dfn[u]) {
                cnt ++;
                dcc_cnt ++;
                if (u != root || cnt >= 2) cut[u] = true;
                int x;
                do {
                    x = stk[top--];
                    dcc[dcc_cnt].push_back(x);
                } while (x != j);
                dcc[dcc_cnt].push_back(u);
            }
        } else {
            low[u] = min(low[u], dfn[j]);
        }
    }
}

int main()
{
    int T = 1;
    while(cin >> m , m)
    {
        for(int i = 1 ; i <= dcc_cnt ; i++) dcc[i].clear();
        idx = n = timestamp = top = dcc_cnt = 0;
        memset(h , -1 , sizeof h);
        memset(dfn , 0 , sizeof dfn);
        memset(cut , 0 , sizeof cut);

        while(m--)
        {
            int a , b;
            cin >> a >> b;
            add(a , b) , add(b , a);
            n = max(n , a) , n  = max(n , b);
        }

        for(root = 1 ; root <= n ; root++)
            if(!dfn[root])
                tarjan(root);

        int res = 0;
        ULL num = 1;
        for(int i = 1 ; i <= dcc_cnt ; i++)
        {
            int cnt = 0;//连通分量中割点数
            for(int j = 0 ; j < (int)dcc[i].size() ; j++)
                if(cut[dcc[i][j]])
                    cnt++;

            if (cnt == 0)
            {
                if (dcc[i].size() > 1) res += 2, num *= dcc[i].size() * (dcc[i].size() - 1) / 2;
                else res ++ ;
            }        
            else if(cnt == 1) res ++ , num *= dcc[i].size() - 1;
        }

        printf("Case %d: %d %llu\n" , T++ , res , num);

    }
    return 0;
}
```

