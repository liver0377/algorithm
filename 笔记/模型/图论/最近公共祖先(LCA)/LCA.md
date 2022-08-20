### 祖孙询问

![image-20220817165702139](http://www.cdn.liver0377.xyz/typora/202208171657197.png)





**解题思路**

- 这里一个裸LCA题目，直接套用模板即可



**代码实现**

```cc
#include <iostream>
#include <algorithm>
#include <cstring>
#include <queue>

using namespace std;

const int N = 4e4 + 10;
const int M = N * 2;

int h[N], e[M], ne[M], idx;
int depth[N], fa[N][16];
int n, root, m;

void add(int a, int b) {
    e[idx] = b, ne[idx] = h[a], h[a] = idx ++;
}

void bfs(int x) {
    memset(depth, 0x3f, sizeof depth);
    queue<int> q;
    q.push(x);
    depth[0] = 0;  // fa[x][0] == 0
    depth[x] = 1;
    
    while (q.size()) {
        int t = q.front(); q.pop();
        
        for (int i = h[t]; ~i; i = ne[i]) {
            int j = e[i];
            if (depth[j] > depth[t] + 1) {
                depth[j] = depth[t] + 1;
                fa[j][0] = t;
                
                for (int k = 1; k <= 15; k++) {
                    fa[j][k] = fa[fa[j][k - 1]][k - 1];
                }
                q.push(j);
            }
        }
    }
}

int lca(int a, int b) {
    if (depth[b] > depth[a]) swap(a, b);
    for (int i = 15; i >= 0; i --) {
        if (depth[fa[a][i]] >= depth[b]) a = fa[a][i];
    }
    if (a == b) return a;
    for (int i = 15; i >= 0; i --) {
        if (fa[a][i] != fa[b][i]) a = fa[a][i], b = fa[b][i];
    }
    return fa[a][0];
}

int main() {
    cin >> n;
    memset(h, -1, sizeof h);
    for (int i = 0; i < n; i++) {
        int a, b;
        cin >> a >> b;
        if (b == -1) root = a;
        else {
            add(a, b);
            add(b, a);
        }
    }
    
    bfs(root);
    
    cin >> m;
    for (int i = 0; i < m; i++) {
        int a, b;
        cin >> a >> b;
        int t = lca(a, b);
        if (t == a) cout << 1 << endl;
        else if (t == b) cout << 2 << endl;
        else cout << 0 << endl;
    }
    
    return 0;
}
```



### 距离

![image-20220818135916143](http://www.cdn.liver0377.xyz/typora/202208181359222.png)



**解题思路**

- 设`dist[i]`表示点`i`到根节点的最短距离, 树上任意两点`x, y`之间的最短距离`d = dist[x] + dist[y] - 2 * dist[LCA(x, y)]`

- `dist`直接通过`bfs`求就行，接下来就变成了一个求`LCA`的问题
- 这里使用`tarjan`算法，在深搜过程中进行查询



**代码实现**

```cc
#include <iostream>
#include <algorithm>
#include <cstring>
#include <vector>
#include <queue>

using namespace std;

const int N = 1e4 + 10;
const int M = N * 2;

typedef pair<int, int> PII;

int h[N], e[M], ne[M], w[M], idx;
int st[N];
int dist[N], p[N], res[M];    // res[i]: 查询i的返回结果

vector<PII> queries[N];    // queries[i].first: 与i有关的查询节点编号
                           // queries[i].second: 查询编号
int n, m;

void bfs(int root, int fa) {
    memset(dist, 0x3f,sizeof dist);
    dist[root] = 0;
    
    queue<int> q;
    q.push(root);
    
    while (q.size()) {
        int t = q.front(); q.pop();
        for (int i = h[t]; ~i; i = ne[i]) {
            int j = e[i];
            if (j == fa) continue;
            if (dist[j] > dist[t] + w[i]) {
                dist[j] = dist[t] + w[i];
                q.push(j);
            }
        }
    }
}

int find(int x) {
    if (x == p[x]) return x;
    return p[x] = find(p[x]);
}

void add(int a, int b, int c) {
    w[idx] = c, e[idx] = b, ne[idx] = h[a], h[a] = idx ++;
}

void tarjan(int x) {
    st[x] = 1;
    for (int i = h[x]; ~i; i = ne[i]) {
        int j = e[i];
        if (st[j]) continue;
        tarjan(j);
        p[j] = x;   
    }
    
    for (auto item : queries[x]) {
        int y = item.first, id = item.second;
        if (st[y] == 2) {
            res[id] = dist[x] + dist[y] - 2 * dist[find(y)];  // 并查集关系在之前的递归中已经求好了
        }
    }
    st[x] = 2;
}

int main() {
    cin >> n >> m;
    
    memset(h, -1, sizeof h);
    for (int i = 0; i < n - 1; i++) {
        int a, b, c;
        cin >> a >> b >> c;
        add(a, b, c);
        add(b, a, c);
    }
    
    for (int i = 0; i < m; i++) {
        int a, b;
        cin >> a >> b;
        if (a == b) continue;
        queries[a].push_back({b, i});
        queries[b].push_back({a, i});
    }
    
    bfs(1, -1);   // 随便选择一个点作为根节点
    
    for (int i = 1; i <= n; i++) p[i] = i;
    tarjan(1);
    
    for (int i = 0; i < m; i++) cout << res[i] << endl;
    return 0;
}
```







### 次小生成树

![image-20220818162505877](http://www.cdn.liver0377.xyz/typora/202208181625940.png)



**解题思路**

- 关于次小生成树有一个定理

  若给定无向图有最小生成树以及严格次小生成树，那么严格次小生成树与最小生成树之间一定只有一条边不同

- 基本流程

  ![image-20220818163128120](http://www.cdn.liver0377.xyz/typora/202208181631162.png)

  ![image-20220818163143822](http://www.cdn.liver0377.xyz/typora/202208181631873.png)

  - 上面步骤中的一个关键子问题是，如何求解环上的最短边权以及次短边权

    假设有如下环

    ![image-20220818163519236](http://www.cdn.liver0377.xyz/typora/202208181635269.png)

    `a`, `b`, `LCA(a, b)`均为最小生成树上面的节点，现在要求的就是红色路径上的最短边权以及次短边权

  - 可以从树根进行`bfs`, 在遍历整棵树的过程中，直接预处理出不同路径的最短边权

    - 设`d1[i][k]`表示从`i`点向上走2^k步的路径中的最大边权, `d2[i][k]`表示从`i`点向上走2^k步的路径中的次大边权

    - 从树根进行`bfs`,当前正在访问的节点为`x`, 可以直接计算从`x`到根节点这条路径上的所有`d1, d2`

    - 可以将整个路径一分为二，进而递归的实现

      设`mid`为$fa[i][k - 1]$

      $d1[i][k] = \{d1[i][k - 1], d1[mid][k - 1], d2[i][k - 1], d2[mid][k - 1]\}$中的最大值

      $d2[i][k] = \{d1[i][k - 1], d1[mid][k - 1], d2[i][k - 1], d2[mid][k - 1]\}$中的次小值

    - 当`k == 0`时，$d1[i][k] = w$, 其中`w`为`i`与父节点之间的边权

**代码实现**

- 时间复杂度: $O(mlog(n))$



```cc
#include <iostream>
#include <algorithm>
#include <cstring>
#include <queue>
#include <set>

using namespace std;

const int N = 1e5 + 10;
const int M = 3e5 * 2 + 10;
const int INF = 0x3f3f3f3f;

typedef long long LL;
struct Edge {
    int a, b, w;
    bool used;      // used: 表示该边是否在最小生成树当中
    bool operator<(const Edge &e) {
        return w < e.w;
    }
} edges[M];

int h[N], e[M], ne[M], w[M], idx;
int p[N], fa[N][17];
int depth[N], d1[N][17], d2[N][17];   // d1[i][k]: 从点i开始向上跳2^k步的路径中的最大边权
                                      // d2[i][k]: 从点i开始向上跳2^k步的路径中的严格次大边权
int n, m;

void add(int a, int b, int c) {
    w[idx] = c, e[idx] = b, ne[idx] = h[a], h[a] = idx ++;
}

int find(int x) {
    if (x == p[x]) return x;
    return p[x] = find(p[x]);
}

LL kruskal() {
  LL res = 0;
  sort(edges, edges + m);
  for (int i = 1; i <= n; i++) p[i] = i;
  for (int i = 0; i < m; i++) {
      auto [a, b, c, _] = edges[i];
      a = find(a);
      b = find(b);
      if (a == b) continue;
      p[a] = b;
      edges[i].used = true;    // 该边被加入最小生成树，顺便标记一下
      res += c;
  }
  return res;
}

void bfs() {
    memset(depth, 0x3f, sizeof depth);
    depth[0] = 0, depth[1] = 1;  // 以1号节点为根节点
    
    queue<int> q;
    q.push(1);
    while (q.size()) {
        int t = q.front(); q.pop();
        for (int i = h[t]; ~i; i = ne[i]) {
            int j = e[i];
            if (depth[j] > depth[t] + 1) {
                depth[j] = depth[t] + 1;
                q.push(j);
                
                // 计算fa, d1[j][k], d2[j][k]
                fa[j][0] = t;
                d1[j][0] = w[i];
                d2[j][0] = -INF;    // 不存在
                for (int k = 1; k <= 16; k ++) {
                    int mid = fa[j][k - 1];
                    fa[j][k] = fa[fa[j][k - 1]][k - 1];
                    int distance[4] = {d1[j][k - 1], d2[j][k - 1], d1[mid][k - 1], d2[mid][k - 1]};
                    for (int x = 0; x < 4; x++) {
                        int dist = distance[x];
                        if (dist > d1[j][k]) {
                            d2[j][k] = d1[j][k];
                            d1[j][k] = dist;
                        } else if (dist != d1[j][k] && dist > d2[j][k]) {
                            d2[j][k] = dist;
                        }
                    }
                }
            }
        }
    }
}

// 将(a, b, w)加入最小生成树之后会产生一个环
// lca()会找到环中的最大/严格次大的某条边(x, y, z)
// 返回w - z

int lca(int a,int b,int w)
{
    static int distance[N * 2];
    int cnt = 0;
    if (depth[a] < depth[b]) swap(a, b); 
    for (int k = 16; k >= 0; k -- )
    
        if (depth[fa[a][k]] >= depth[b])
        {
            distance[cnt ++ ] = d1[a][k];
            distance[cnt ++ ] = d2[a][k];
            a = fa[a][k];
        }
    if (a != b)
    {
        for (int k = 16; k >= 0; k -- )
            if (fa[a][k] != fa[b][k])
            {
                distance[cnt ++ ] = d1[a][k];
                distance[cnt ++ ] = d2[a][k];
                distance[cnt ++ ] = d1[b][k];
                distance[cnt ++ ] = d2[b][k];
                a = fa[a][k], b = fa[b][k];
            }
        // 此时a和b到lca下同一层 所以还要各跳1步=跳2^0步
        distance[cnt ++ ] = d1[a][0];
        distance[cnt ++ ] = d1[b][0];
    }
    // 找a,b两点距离的最大值dist1和次大值dist2
    int dist1 = -INF, dist2 = -INF;
    for (int i = 0; i < cnt; i ++ )
    {
        int d = distance[i];
        if (d > dist1) dist2 = dist1, dist1 = d;
        else if (d != dist1 && d > dist2) dist2 = d;
    }
    
    if (w > dist1) return w - dist1;
    // 否则w==dist1 w替换dist2
    if (w > dist2) return w - dist2;
}

void build() {
    memset(h, -1, sizeof h);
    for (int i = 0; i < m; i++) {
        auto [a, b, c, u] = edges[i];
        if (u) {
            add(a, b, c), add(b, a, c);
        }
    }
}

int main() {
    cin >> n >> m;
    for (int i = 0; i < m; i++) {
        int a, b, c;
        cin >> a >> b >> c;
        edges[i] = {a, b, c};
    }
    LL sum = kruskal();    // 计算出最小生成树边权和
    build();               // 建立最小生成树
    bfs();                 // 计算fa以及d1, d2
    LL res = 1e18;
    // 枚举所有非树边
    for (int i = 0; i < m; i ++) {
        if (edges[i].used) continue;
        auto [a, b, c, _] = edges[i];
        res = min(res, sum + lca(a, b, c));
    }
    
    cout << res << endl;
    return 0;
}
```









### 暗的连环

![image-20220818175658531](http://www.cdn.liver0377.xyz/typora/202208181756600.png)

![image-20220818175712846](http://www.cdn.liver0377.xyz/typora/202208181757901.png)





**解题思路**

- 题意转换

  给定一张无向图，其中有`n`个主要边以及`m`个附加边，`n`个主要边所连接的节点构成一棵生成树

  现要选择其中的一个主要边，再选择其中的一个附加边，使得去除这两个边之后，整张图一分为二，变为两个连通块

- 对于一棵由主要边构成的生成树，如果加上一条附加边，形成的新图中就会包含环，如果想要将新图一分为二，就必须选择环上的

  任意一条主要边以及环上唯一的附加边

- 可以考虑给边设置权值，每加入一条附加边，就对环上的所有主要边的权值+1

  ![image-20220818180304514](http://www.cdn.liver0377.xyz/typora/202208181803560.png)

- 只要选择图上任意一个环上的任意一条边权为1的主要边，以及该环上唯一的附加边，就能砍断该树

  该做法中，对于每一对附加边(a, b)， 都要对整个环上的边进行操作，可以采取**树上差分**的形式来简化边权操作

- 设`d[i]`表示点`i`的权值，`pi`为`i`的父亲，对于(pi, i)这条边来说，其权值为以`i`的根节点的子树上的所有节点的权值之和

- 每次加入一个附加边(x, y), 可以令`d[x] ++, d[y] ++, d[lca(x, y)] -= 2`

  ![image-20220818181315508](http://www.cdn.liver0377.xyz/typora/202208181813573.png)

  - 首先对于子树外部的节点来说，整个树的节点权值之和没有变，那么外面的所有边的权值也不会改变

  - 对于子树内部而言，所有以`x`, `y`为子节点的节点`k`所对应的边的权值(pk, k)都会加1

    这样就快速完成了对所有环上的边的边权+1的操作

- 预处理出所有的`fa`, `depth`之后，可以使用`dfs`遍历所有的生成树节点(主要边)

  设任意一条边权为`k`

  - 若`k == 0`, 则砍断该主要边之后，可以任意砍断一条附加边, ans += m
  - 若`k == 1`, 则砍断该主要边之后，必须砍断环上的附加边, ans ++
  - 若`k > 1`, 则无法砍断该主要边使得整张图一份为2，因此什么也不做



**代码实现**

```cc
#include <iostream>
#include <algorithm>
#include <cstring>
#include <queue>

using namespace std;

const int N = 1e5 + 10;
const int M = N * 3;

int h[N], e[M], ne[M], idx;
int d[N], depth[N], fa[N][17];     // d[i]: 点i的权值
int n, m, res;

void add(int a, int b) {
    e[idx] = b, ne[idx] = h[a], h[a] = idx ++;
}

void bfs() {
    memset(depth, 0x3f, sizeof depth);
    depth[0] = 0, depth[1] = 1;
    queue<int> q;
    q.push(1);
    
    while (q.size()) {
        int t = q.front(); q.pop();
        for (int i = h[t]; ~i; i = ne[i]) {
            int j = e[i];
            if (depth[j] > depth[t] + 1) {
                depth[j] = depth[t] + 1;
                q.push(j);
                
                fa[j][0] = t;
                for (int k = 1; k <= 16; k++) {
                    fa[j][k] = fa[fa[j][k - 1]][k - 1];
                }
            }
        }
    }
}

int lca(int a, int b) {
    if (depth[a] < depth[b]) swap(a, b);
    for (int k = 16; k >= 0; k--) {
        if (depth[fa[a][k]] >= depth[b]) a = fa[a][k];
    }
    if (a == b) return a;
    for (int k = 16; k >= 0; k--) {
        if (fa[a][k] != fa[b][k]) {
            a = fa[a][k];
            b = fa[b][k];
        }
    }
    return fa[a][0];
}

// 返回以u为根节点的子树当中所有节点的权值之和，也就是(p[a], u)这条边的权值
int dfs(int u, int father) {
    int sum = d[u];
    for (int i = h[u]; ~i; i = ne[i]) {
        int j = e[i];
        if (j == father) continue;
        int s = dfs(j, u); 
        sum += s;
        if (s == 0) res += m;
        else if (s == 1) res ++;
    }
    return sum;
}

int main() {
    cin >> n >> m;
    memset(h, -1, sizeof h);
    for (int i = 0; i < n - 1; i++) {
        int a, b;
        cin >> a >> b;
        add(a, b), add(b, a);
    }
    
    bfs();   // 求fa以及depth
    
    for (int i = 0; i < m; i++) {
        int a, b;
        cin >> a >> b;
        int p = lca(a, b);
        d[a] ++, d[b] ++, d[p] -= 2;   // 书上差分
    }
    
    dfs(1, -1);
    
    cout << res << endl;
    return 0;
}
```

