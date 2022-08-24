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
      stk[++ top] = u, in_stk[u] = true;
      
      for (int i = h[u]; ~i; i = ne[i]) {
          int j = e[i];
          if (!dfn[j]) {
              tarjan(j);
              low[u] = min(low[u], low[j]);
          } else if (in_stk[j]) {
              low[u] = min(low[u], dfn[j]);
          }
      }
      
      if (dfn[u] == low[u]) {
          scc_cnt ++;
          int x;
          do {
              x = stk[top --];
              in_stk[x] = false;
              id[x] = scc_cnt;
              sz[scc_cnt] ++;
          } while (x != u);
      }
   }
  ```



**拓扑序**

- 当使用`tarjan`算法求完强连通分量时，`scc_id`越大的节点，排在拓扑序的越前面

  按照逆序遍历`scc_id`,就是按照拓扑序访问节点



**DAG与SCC**

- 对于一张DAG而言，设其入度为0的节点个数为P, 出度为0的节点个数为Q, 则若想要将该DAG转换为一个强连通图

  需要至少添加$P + Q$条边



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

![image-20220822170904381](http://www.cdn.liver0377.xyz/typora/202208221709455.png)

![image-20220822170916366](http://www.cdn.liver0377.xyz/typora/202208221709406.png)



**解题思路**

- 一个连通图一定是一个半连通图，多个连接分量所形成的一条链也是一个半连通图

  > 对于链上的任意两个点，他们之间一定有一条路径

- 首先对原图进行缩点，**建立一张缩点之后的新图**, 这里真的建立新图是因为要遍历这张图

- 接着按照拓扑序(逆scc序)遍历新图，设`f[i]`表示从起点到连通分量`i`的链上的最大节点数， `g[i]`表示取得最大节点数的路径条数(半联通子图数目)

  每遍历一个节点，就用它更新其后面节点的`f`, `g`

- 最终枚举最大的`f`以及对应的`g`即可



```cc
#include <iostream>
#include <algorithm>
#include <cstring>
#include <unordered_set>

using namespace std;

const int N = 1e5 + 10;
const int M = 2e6 + 10;


typedef long long LL;

int n, m, top, timestamp, scc_cnt, mod;
int h[N], e[M], ne[M], idx, hs[N];    // hs: 供DAG新图使用
int dfn[N], low[N], id[N], sz[N], stk[N];
int f[N], g[N];        // f[i]: 按照拓扑序访问，以强连通块i为终点的链上的最大节点总数
                       // g[i]: 强连通块取最大节点总数的取法
bool in_stk[N];


void add(int h[], int a, int b) {
    e[idx] = b, ne[idx] = h[a], h[a] = idx ++;
}

void tarjan(int u) {
    dfn[u] = low[u] = ++ timestamp ;
    stk[++ top] = u, in_stk[u] = true;
    
    for (int i = h[u]; ~i; i = ne[i]) {
        int j = e[i];
        if (!dfn[j]) {
            tarjan(j);
            low[u] = min(low[u], low[j]);
        } else if (in_stk[j]) {
            low[u] = min(low[u], dfn[j]);
        }
    }
    
    if (dfn[u] == low[u]) {
        scc_cnt ++;
        int x;
        do {
            x = stk[top --];
            in_stk[x] = false;
            id[x] = scc_cnt;
            sz[scc_cnt] ++;
        } while (x != u);
    }
 }

int main() {
    cin >> n >> m >> mod;
    memset(h, -1, sizeof h);
    memset(hs, -1, sizeof hs);
    for (int i = 0; i < m; i++) {
        int a, b;
        cin >> a >> b;
        add(h, a, b);
    }
    
    // 1. 缩点
    for (int i = 1; i <= n; i++) {
        if (!dfn[i]) {
            tarjan(i);
        }
    }
    
    // 2. 建立DAG新图
    unordered_set<LL> st;
    for (int i = 1; i <= n; i++) {
        for (int j = h[i]; ~j; j = ne[j]) {
            int k = e[j];
            int a = id[i], b = id[k];
            LL hash = a * 1000000ll + b;
            if (a != b && !st.count(hash)) {   // 这里的hash为去重操作，避免两个连通块之间连接多条边
                                               // 因为两个半连通子图之间如果只有边不同，而顶点相同，则算
                                               // 同一个半连通子图
                add(hs, a, b);
                st.insert(hash);
            }
        }
    }
    
    // 3. 按照拓扑序递推f[], g[]
    for (int i = scc_cnt; i >= 1; i--) {
        if (!f[i]) {
            f[i] = sz[i];
            g[i] = 1;
        } 
            for (int j = hs[i]; ~j; j = ne[j]) {
                int k = e[j];
                int t = sz[k];
                if (f[k] < f[i] + t) {
                    f[k] = f[i] + t;
                    g[k] = g[i];
                } else if (f[k] == f[i] + t) {
                    g[k] = (g[k] + g[i]) % mod;
                }
            }
        
    }
    
    // 4. 求最大f[i]以及对应的g[i]
    int maxf = 0, maxg = 0;
    for (int i = 1; i <= scc_cnt; i ++) {
        if (f[i] > maxf) {
            maxf = f[i];
            maxg = g[i];
        } else if (f[i] == maxf) {
            maxg = (maxg + g[i]) % mod;
        }
    }
    
    cout << maxf << endl;
    cout << maxg << endl;
    return 0;
}
```







### 银河

![image-20220822184554441](http://www.cdn.liver0377.xyz/typora/202208221845520.png)



**解题思路**

- 此题与[糖果](https://www.acwing.com/problem/content/1171/)一模一样，但是会卡`spfa`， 这里使用`tarjan`算法求解
- 思路是一样的，依旧是求最长路，首先使用`tarjan`算法进行缩点，
- 在连通分量内部，所有的边都位于环上，因此只要有一条边的边权大于0，那么就表明整张图有正环，即无解
- 如果有解的话，整个连通分量内部，任意两点之间的最长路应该都是相同的，因此可以直接将整个连通分量看做一个点
- 建立新图之后，按照拓扑序对新图进行动态规划递推，求每个连通分量到超级源点的最长最长距离
- 最终进行累加的时候，每个连通分量内部的亮度总和就是`dist[i] * sz[i]`



**代码实现**

```cc
#include <iostream>
#include <algorithm>
#include <cstring>

using namespace std;

typedef long long LL;
const int N = 1e5 + 10;
const int M = 6e5 + 10;

int dist[N];
int h[N], hs[N], e[M], ne[M], w[M], idx;
int dfn[N], low[N], timestamp;
int stk[N], top;
bool in_stk[N];
int id[N], sz[N], scc_cnt;
int n, m;

void add(int h[], int a, int b, int c) {
    w[idx] = c, e[idx] = b, ne[idx] = h[a], h[a] = idx ++;
}

void tarjan(int u) {
    dfn[u] = low[u] = ++ timestamp ;
    stk[++ top] = u, in_stk[u] = true;
    
    for (int i = h[u]; ~i; i = ne[i]) {
        int j = e[i];
        if (!dfn[j]) {
            tarjan(j);
            low[u] = min(low[u], low[j]);
        } else if (in_stk[j]) {
            low[u] = min(low[u], dfn[j]);
        }
    }
    
    if (dfn[u] == low[u]) {
        scc_cnt ++;
        int x;
        do {
            x = stk[top --];
            in_stk[x] = false;
            id[x] = scc_cnt;
            sz[scc_cnt] ++;
        } while (x != u);
    }
 }

int main() {
    cin >> n >> m;
    memset(h, -1, sizeof h);
    memset(hs, -1, sizeof hs);
    // 1. 建图
    for (int i = 1; i <= n; i++) add(h, 0, i, 1);  // 超级源点0
    for (int i = 0; i < m; i++) {
        int T, a, b;
        cin >> T >> a >> b;
        if (T == 1) {
            add(h, a, b, 0);
            add(h, b, a, 0);
        } else if (T == 2) {
            add(h, a, b, 1);
        } else if (T == 3) {
            add(h, b, a, 0);
        } else if (T == 4) {
            add(h, b, a, 1);
        } else {
            add(h, a, b, 0);
        }
    }
    
    
    // 2. 缩点
    tarjan(0);    // 从0点可以到达所有点
    
    // 3. 建新图, 顺便判断一下连通分量内部是否有边权为1的边
    bool flag = false;
    for (int i = 0; i <= n; i++) {
        for (int j = h[i]; ~j; j = ne[j]) {
            int k = e[j];
            int a = id[i], b = id[k];
            if (a != b) {
                add(hs, a, b, w[j]);
            } else if (w[j] > 0) {
                flag = true;
                goto FAIL;
            }
        }
    }
    
FAIL:
    // 4. 按照拓扑序递推
    if (flag) {
        cout << -1 << endl;
    } else {
        for (int i = scc_cnt; i; i --) {
            for (int j = hs[i]; ~j; j = ne[j]) {
                int k = e[j];
                dist[k] = max(dist[i] + w[j], dist[k]);
            }
        }
        LL res = 0;
        for (int i = 1; i <= scc_cnt; i ++) {
            res += (LL)sz[i] * dist[i];
        }
        cout << res << endl;
    }
    return 0;
}
```

> 这里连通分量之间是可以有重边的，在求最长路时会自然地选择较长边，不需要去重
