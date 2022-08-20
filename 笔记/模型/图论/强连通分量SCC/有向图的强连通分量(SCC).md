- 强连通图

  在一个有向图中，对于任意两个点`u`, `v`, 从`u`可以到达`v`, 也可以从`v`到达`u`, 那么该图就是强连通图

- 强连通分量

  强连通分量是指一个有向图的极大强连通子图，若在极大强连通子图上加上任意一个点，该图都不是强连通子图

- 流图

  ![image-20220820141443606](http://www.cdn.liver0377.xyz/typora/202208201414676.png)

  - 搜索树

    ![image-20220820141517101](http://www.cdn.liver0377.xyz/typora/202208201415135.png)

  - 时间戳

    ![image-20220820141539048](http://www.cdn.liver0377.xyz/typora/202208201415087.png)

  - 边的分类

    ![image-20220820141555381](http://www.cdn.liver0377.xyz/typora/202208201415429.png)

    > 可以发现，树枝边是一种特殊的前向边



### tarjan(SCC)

`tarjan`算法可以对图进行**缩点**，也就是将每个强连通分块变为一个点，最终整张图可以变为一个DAG, 对于DAG来说，很多操作都比其它图要简单



- `low[u]`: 在以`u`为根的**搜索树**当的所有子节点以及`u`自身构成的节点集合当中任选一个点为起点，能够到达当前的栈中的点的最小`dfn`

  - 例

  ![image-20220820151833453](http://www.cdn.liver0377.xyz/typora/202208201518487.png)

  在上图中, `[]`内部包含的即为该节点的`low`值，节点8的子树集合为{8, 9}, 其中从节点8可以走到最小的`dfn`为6，并且6在栈当中， 因此`low[8] = 6`

  

- `tarjan`算法流程

  `tarjan`算法基于`dfs`,需要手动维护一个栈，用于记录当前可能的连通分块中的点

  - 从节点`u`出发，将`u`入栈，设置`dfn[u] = low[u] = timestamp`

  - 遍历`u`的每一条出边(u, v)

    - 若`v`没有访问过，则表明其应该属于`u`的子树，递归执行`tarjan(v)`, 求出以`v`为根的子树的所有节点的`low`, `dfn`之后，尝试更新`low[u]`

      `low[u] = min(low[u], low[v])`

    - 若`v`访问过，并且在栈当中，则表明`v`正处于当前的连通分块中，尝试更新`low[u]`

      `low[u] = min(low[u], lov[v])`

    - 若`v`访问过，并且不在栈当中，说明`v`已经被加入另一个连通分块中了，此时什么也不做

  - 当所有出边都已经遍历完毕，回溯到`u`时，若有`dfn[u] == low[u]`,  则表明有环，也就是当前栈中, 从元素`u`到栈顶元素`x`的所有元素构成了一个强连通分量，此时将`u`到`x`以及它们之间的所有元素出栈，强连通分量总数+1

- 算法模板

  ```cc
  void tarjan(int u) {
      dfn[u] = low[u] = ++ timestamp ;
      stk[++ top] = u, in_stack[u] = true;
      
      for (int i = h[u]; ~i; i = ne[i]) {
          int j = e[i];
          if (!dfn[j]) {
              tarjan(j);
              low[u] = min(low[u], low[j]);
          } else if (in_stack[j]) {
              low[u] = min(low[u], dfn[j]);
          }
      }
      
      if (dfn[u] == low[u]) {
          scc_cnt ++;
          int x;
          do {
              x = stk[top --];
              in_stack[x] = false;
              id[x] = scc_cnt;
              sz[scc_cnt] ++;
          } while (x != u);
      }
   }
  ```



**拓扑序**

- 当使用`tarjan`算法求完强连通分量时，`scc_id`越大的节点，排在拓扑序的越前面

  按照逆序遍历`scc_id`,就是按照拓扑序访问节点



### 受欢迎的牛

![image-20220820163636325](http://www.cdn.liver0377.xyz/typora/202208201636382.png)



**解题思路**

- 题意转换

  给定一张有向图，求有多少个节点满足从其它所有节点都可以到达该节点

- 可以建立一张反图，然后使用与一遍`bfs`，但是这样做的时间复杂度是$O(n^2)$

- 可以对该图进行缩点，使得该图变为一张有向无环图(DAG), 在DAG中，若有2个或以上的节点的出度为0，那么显然在这些节点之间

  无法从任意一个节点走到另外一个节点

- 如果该DAG只有一个出度为0的节点，那么该点所在的最大连通分量(SCC)中的所有点都满足要求，因为SCC中的所有点都是互相连通的

- 因此，首先要判断缩点之后的DAG中是否只有一个出度为0的点，如果是，返回该点所在的强连通分量的点的数目，否则，返回0

  > 使用tarjan算法进行缩点

**代码实现**

- 时间复杂度: O(n)

```cc
#include <iostream>
#include <algorithm>
#include <cstring>

using namespace std;

const int N = 1e4 + 10;
const int M = 5e4 + 10;

int h[N], e[M], ne[M], idx;
int dfn[N], low[N], stk[N], top;
bool in_stack[N];
int sz[N], id[N], dout[N];       // id[i]: 节点i所在的连通块，从1开始
                                 // sz[i]: 连通块i的节点数目
                                 // dout[i]: 连通块i的出度
int n, m, timestamp, scc_cnt;

void tarjan(int u) {
    dfn[u] = low[u] = ++ timestamp ;
    stk[++ top] = u, in_stack[u] = true;
    
    for (int i = h[u]; ~i; i = ne[i]) {
        int j = e[i];
        if (!dfn[j]) {
            tarjan(j);
            low[u] = min(low[u], low[j]);
        } else if (in_stack[j]) {
            low[u] = min(low[u], dfn[j]);
        }
    }
    
    if (dfn[u] == low[u]) {
        scc_cnt ++;
        int x;
        do {
            x = stk[top --];
            in_stack[x] = false;
            id[x] = scc_cnt;
            sz[scc_cnt] ++;
        } while (x != u);
    }
}

void add(int a, int b) {
    e[idx] = b, ne[idx] = h[a], h[a] = idx ++;
}

int main() {
    cin >> n >> m;
    memset(h, -1, sizeof h);
    for (int i = 0; i < m; i++) {
        int a, b;
        cin >> a >> b;
        add(a, b);
    }
    
    // 可能为非连通图，因此直接枚举所有节点为根节点
    // tarjan算法对于访问过并且不在栈中的节点是无视的，因此不会多算连通块
    for (int i = 1; i <= n; i++) {
        if (!dfn[i]) {
            tarjan(i);
        }
    }
    
    for (int i = 1; i <= n; i++) {
        for (int j = h[i]; ~j; j = ne[j]) {
            int k = e[j];
            int a = id[i], b = id[k];
            if (a == b) continue;
            dout[a] ++;
        }
    }
    
    int sum = 0, cnt = 0;
    for (int i = 1; i <= scc_cnt; i++) {
        if (!dout[i]) {
            cnt ++;
            if (cnt == 2) {
                sum = 0;
                break;
            } 
            sum += sz[i];
        }
    }
    cout << sum << endl;
    return 0;
}
```











### 学校网络

![image-20220820174730022](http://www.cdn.liver0377.xyz/typora/202208201747077.png)

![image-20220820174743677](http://www.cdn.liver0377.xyz/typora/202208201747725.png)



**解题思路**

- 题意转化

  给定一张有向图，若选择任意一个节点，就可以覆盖该节点所能够到达的所有节点，求

  1. 至少要选择多少节点才能够覆盖所有节点
  2. 至少要添加多少条新边才能够使得整张图变为强连通图

- 为了简化该题，首先可以使用`tarjan`算法对该图进行缩点

- 问题1：至少要选择多少节点才能够覆盖所有节点

  设缩点之后形成的DAG中有`P`个起点, `Q`个终点, 那么只要选择所有`P`个起点，就一定能够覆盖所有的终点，因此答案是`P`

- 问题2： 至少要添加多少条新边才能够使得整张图变为强连通图

  - 结论: `max(P, Q)`

  - 证明:

    ![image-20220820175431808](http://www.cdn.liver0377.xyz/typora/202208201754873.png)

    故答案为`max(P, Q)`

    注意，当图中只有强连通分量时，也就是整张图为强连通图时，直接返回0即可

**代码实现**

```cc
#include <iostream>
#include <algorithm>
#include <cstring>

using namespace std;

const int N = 110;
const int M = N * N;

int h[N], e[M], ne[M], idx;
int dfn[N], low[N], stk[N], id[N];
bool in_stack[N];
int din[N], dout[N];          // din[i]: 连通块i的入度
                              // dout[i]: 连通块i的出度
int n, scc_cnt, timestamp, top;

void tarjan(int u) {
    dfn[u] = low[u] = ++timestamp;
    stk[++top] = u, in_stack[u] = true;
    for (int i = h[u]; ~i; i = ne[i]) {
        int j = e[i];
        if (!dfn[j]) {
            tarjan(j);
            low[u] = min(low[u], low[j]);
        } else if (in_stack[j]) {
            low[u] = min(low[u], dfn[j]);
        }
    }
    if (dfn[u] == low[u]) {
        ++scc_cnt;
        int v;
        do {
            v = stk[top--];
            in_stack[v] = false;
            id[v] = scc_cnt;
        } while (v != u);
    }
} 


void add(int a, int b) {
    e[idx] = b, ne[idx] = h[a], h[a] = idx ++;
}

int main() {
    cin >> n;
    
    memset(h, -1, sizeof h);
    for (int i = 0; i < n; i++) {
        int x;
        while (cin >> x, x) {
            // cout << "add(" << i + 1 << ", " << x << ")" << endl;
            add(i + 1, x);
        }
    }
    
    for (int i = 1; i <= n; i++) {
        if (!dfn[i])
            tarjan(i);
    }
    
    // cout << scc_cnt << endl;
    for (int i = 1; i <= n; i++) {
        for (int j = h[i]; ~j; j = ne[j]) {
            int k = e[j];
            int a = id[i], b = id[k];
            if (a == b) continue;
            dout[a] ++, din[b] ++;
        }
    }
    
    int p = 0, q = 0;
    for (int i = 1; i <= scc_cnt; i++) {
        if (!din[i]) p++;
        if (!dout[i]) q ++;
    }
    
    cout << p << endl;
    if (scc_cnt == 1) cout << 0 << endl;
    else cout << max(p, q) << endl;
    return 0;
}
```







### 最大半连通子图



```cc
#include <cstdio>
#include <cstring>
#include <iostream>
#include <algorithm>
#include <unordered_set>

using namespace std;

typedef long long LL;

const int N = 100010, M = 2000010;

int n, m, mod;
int h[N], hs[N], e[M], ne[M], idx;
int dfn[N], low[N], timestamp;
int stk[N], top;
bool in_stk[N];
int id[N], scc_cnt, scc_size[N];
int f[N], g[N];
// 有新图所以传h
void add(int h[],int a,int b)
{
    e[idx] = b,ne[idx] = h[a],h[a] = idx++;
}

void tarjan(int u)
{
    dfn[u] = low[u] = ++timestamp;
    stk[++top] = u,in_stk[u] = true;
    for(int i =h[u];~i;i=ne[i])
    {
        int j = e[i];
        // 树枝边
        if(!dfn[j])
        {
            tarjan(j);
            low[u] = min(low[u],low[j]);
        }
        // 横向边
        else if(in_stk[j])low[u] = min(low[u],dfn[j]);
    }
    // 当前分量最高点 把当前点取出来作为一个强连通分量
    // ⭐
    // 解释一下为什么tarjan完是逆dfs序
    // 假设这里是最高的根节点fa
    // 上面几行中 fa的儿子节点j都已经在它们的递归中走完了下面9行代码
    // 其中就包括 ++scc_cnt 
    // 即递归回溯到高层节点的时候 子节点的scc都求完了
    // 节点越高 scc_id越大
    // 在我们后面想求链路dp的时候又得从更高层往下
    // 所以得for(int i=scc_cnt(根节点所在的scc);i;i--)开始
    if(dfn[u]==low[u])
    {
        ++scc_cnt;
        int y;
        do{//由于是dfs搜到一层stk.push 所以stk最先pop出来的是最深层的
            y=stk[top--];
            in_stk[y] = false;
            // id[y] = scc_cnt 属于第scc_cnt个强连通分量
            id[y] = scc_cnt;
            scc_size[scc_cnt]++;
        }while(y!=u);
    }
}

int main()
{
    memset(h,-1,sizeof h);
    memset(hs,-1,sizeof hs);
    cin >> n >> m >> mod;
    while(m--)
    {
        int a,b;
        cin >> a >> b;
        add(h,a,b);
    }
    // tarjan算强连通分量
    for(int i = 1;i<=n;i++)
    {
        if(!dfn[i])
        {
            tarjan(i);
        }
    }
    unordered_set<LL> S;// 边是(u,v) hash后 u*1000000+v
    for(int i=1;i<=n;i++)
    {
        for(int j = h[i];~j;j=ne[j])
        {
            int k = e[j];
            int a = id[i],b= id[k];
            // 边判重
            LL hash = a*1000000ll+b;
            // 如果a和b不在一个强连通分量 且 边(a,b)没被加过
            if(a!=b && !S.count(hash))
            {
                add(hs,a,b);
                S.insert(hash);
            }
        }
    }
    // 强连通分量算完后
    // 拓扑序一定是按照节点编号递减的顺序->不需要重新拓扑排序了
    // 在这个拓扑序上求最长路
    for(int i=scc_cnt;i;i--)
    {
        // !f[i] i没被更新过 i为一条链的起点
        if(!f[i])
        {
            f[i] = scc_size[i];
            g[i] = 1;
        }
        for(int j = hs[i];~j;j=ne[j])
        {
            int k = e[j];
            if(f[k]<f[i]+scc_size[k])
            {
                f[k] = f[i]+scc_size[k];
                g[k] = g[i];
            }
            else if(f[k]==f[i]+scc_size[k])
            {
                g[k] = (g[k]+g[i])%mod;
            }
        }
    }

    int maxf = 0,sum = 0;
    // 对比每一条路终点
    for(int i = 1;i<=scc_cnt;i++)
    {
        if(f[i]>maxf)
        {
            maxf = f[i];
            sum = g[i];
        }
        else if(f[i] == maxf)sum=(sum+g[i])%mod;
    }
    cout << maxf << endl;
    cout << sum;
    return 0;
}
```

