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

