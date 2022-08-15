- 负环定义

  若图中存在一个环，环上各个边的权值之和为负数，那么该环被称为负环



如果图中有负环存在，那么在使用`BellmanFord`算法以及`spfa`算法时，就永远不能够结束，因为它们是根据三角不等式

$dist[y] = dist[x] + z$收敛来判断最短路的，在负环上，总存在有向边(x, y, z)使得该不等式成立



- 判断负环的方法

  有两种判断负环的方法

  - BellmanFord

    ![image-20220815144421988](http://www.cdn.liver0377.xyz/typora/202208151444036.png)

  - spfa

    ![image-20220815144438509](http://www.cdn.liver0377.xyz/typora/202208151444541.png)

  `BellmanFord`算法的复杂度是O(nm), `spfa`则是O(km), 最坏情况下可能会达到$O(m)$, 因此，**通常使用spfa判断负环**



- `spfa`求负环的特殊处理

  有时，可能图不是一个连通图，从起点可能无法走到负环上，也就无法更新负环点上的`dist`了, `spfa`算法会失效，可以采取一种通用的做法来避免这种问题

  - 首先将所有点加入队列
  - 将所有点的`dist`设置为0

  可以从另一个角度来解释这种做法：

     在图中建立一个虚拟节点，其与所有节点都有一条边权为0的边，若要判断原图有没有负环，

  只需要判断这个新图有没有负环即可，从虚拟起点出发，虚拟节点的`dist`为0, 其它节点的`dist`为正无穷，使用`spfa`算法，第一步

  就会更新所有节点的`dist`, 然后将所有除虚拟节点之外地节点加入队列，此时的效果就等价于上面给出的操作

  > 实际上，初始化时点的`dist`随便给多少都行，这中间有一个证明过程，这里就省略了

- Trick
  - `spfa`算法有时复杂度回来到$O(nm)$, 可能会超时
  - 可以通过预判的方式来提早判断是否有负环，当然并不一定保证正确性
  - 通常使用的方法是**若入队次数 >= 2n**, 则认为负环存在







### 虫洞

![image-20220815153849793](http://www.cdn.liver0377.xyz/typora/202208151538867.png)

![image-20220815153906846](http://www.cdn.liver0377.xyz/typora/202208151539896.png)



**解题思路**

- 题意转换

  给定一张有向图，其中有`2m`条单向边, `w`条单向边，求其中是否有负环

- 一个模板题，直接使用`spfa`求负环模板即可



**代码实现**

```cc
#include <iostream>
#include <algorithm>
#include <cstring>
#include <queue>

using namespace std;

const int N = 510;
const int M = 2510 * 2;
const int W = 210;

int h[N], e[M + W], ne[M + W], w[M + W], idx;
int cnt[N], dist[N];
bool st[N];
int T, n, m, s;

bool spfa() {
    memset(dist, 0, sizeof dist);
    memset(cnt, 0, sizeof cnt);
    memset(st, 0, sizeof st);
    
    queue<int> q;
    for (int i = 1; i <= n; i++) {
        q.push(i);
        st[i] = true;
    }
    
    while (q.size()) {
        int t = q.front(); q.pop();
        st[t] = false;
        
        for (int i = h[t]; ~i; i = ne[i]) {
            int j = e[i];
            if (dist[j] > dist[t] + w[i]) {
                cnt[j] = cnt[t] + 1;
                if (cnt[j] >= n) return true;
                dist[j] = dist[t] + w[i];
                if (!st[j]) {
                    q.push(j);
                    st[j] = true;
                }
            }
        }
    }
    return false;
}

void add(int a, int b, int c) {
    w[idx] = c, e[idx] = b, ne[idx] = h[a], h[a] = idx ++;
}

int main() {
    cin >> T;
    while (T --) {
        cin >> n >> m >> s;
        memset(h, -1, sizeof h);
        idx = 0;
        for (int i = 0; i < m; i++) {
            int a, b, c;
            cin >> a >> b >> c;
            add(a, b, c);
            add(b, a, c);
        }
        
        for (int i = 0; i < s; i++) {
            int a, b, c;
            cin >> a >> b >> c;
            add(a, b, -c);
        }
        
        if (spfa()) {
            cout << "YES" << endl;
        } else {
            cout << "NO" << endl;
        }
    }
    return 0;
}
```











### 观光奶牛

![image-20220815165300366](http://www.cdn.liver0377.xyz/typora/202208151653431.png)



**解题思路**

- 这是一个01分数规划问题，可以看



**代码实现**

```cc
#include <iostream>
#include <algorithm>
#include <cstring>
#include <iomanip>
#include <queue>

using namespace std;

const int N = 1010;
const int M = 5010;

int h[N], e[M], ne[M], idx;
int f[N], wt[M], cnt[N];
double dist[N];
bool st[N];
int n, m;

void add(int a, int b, int c) {
    wt[idx] = c, e[idx] = b, ne[idx] = h[a], h[a] = idx ++;
}

bool check(double mid) {
    memset(dist, 0, sizeof dist);
    memset(cnt, 0, sizeof cnt);
    memset(st, 0, sizeof st);
    
    queue<int> q;
    for (int i = 1; i <= n; i++) {
        q.push(i);
        st[i] = true;
    }
    while (q.size()) {
        int t = q.front(); q.pop();
        st[t] = false;
        
        for (int i = h[t]; ~i; i = ne[i]) {
            int j = e[i];
            if (dist[j] > dist[t] + wt[i] * mid - f[t]) {
                dist[j] = dist[t] + wt[i] * mid - f[t];
                cnt[j] = cnt[t] + 1;
                if (cnt[j] >= n) return true;
                if (!st[j]) {
                    q.push(j);
                    st[j] = true;
                }
            }
        }
    }
    return false;
}

int main() {
    cin >> n >> m;
    for (int i = 1; i<= n; i++) cin >> f[i];
    memset(h, -1, sizeof h);
    for (int i = 0; i < m; i++) {
        int a, b, c;
        cin >> a >> b >> c;
        add(a, b, c);
    }
    
    double l = 0.0, r = 1000.0;
    while (r - l > 1e-4) {
        double mid = (l + r) / 2;
        if (check(mid)) l = mid;
        else r = mid;
    }
    cout << std::fixed << setprecision(2) << l << endl;
    return 0;
}
```



