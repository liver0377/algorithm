### 二次扫描与换根法

二次扫描与换根法通常用于解决**不定根**问题, 这种题目的特点是, 给定一个树形结构, 需要以每个节点为根, 进行一系列统计

- 流程

  - 通常题目给定的会是一个无向图

  - 首先任意选择一个节点为根节点, 然后将整个图当做一个有向图, 进行树形DP, 计算出根节点的某个状态

  - 最终的答案数组往往满足这样一个性质: **子节点的状态可以从父节点递推到**

    因此, 可以从根节点进行`dfs`, 自顶向下地进行递推



#### 积蓄水流

![image-20221007085615627](http://www.cdn.liver0377.xyz/typora/202210070856675.png)

![image-20221007085623377](http://www.cdn.liver0377.xyz/typora/202210070856415.png)



**解题思路**

- 设`d[i]`表示以节点`i`为根的子树, 将其当做源点, 从`i`出发, 流向所有子树的流量的最大和为多少

  设`f[i]`表示以节点`i`为根的树, 将其当做源点, 无视有向图的方向限制, 流向其它所有节点的流量的最大总和

  

- 若`x`为父节点, `y`为其某个子节点, `c(x, y)`表示`x`到`y`这条边的流量限制, 则`d[x]`满足

  ![image-20221007090326239](http://www.cdn.liver0377.xyz/typora/202210070903271.png)

- 若`x`为父节点, `y`为其某个子节点, 则`f[y]`由两个部分组成

  - 流向`y`的子树的的流量, 这部分之前已经算过了, 就是`d[y]`

  - 流向`x`的流量

    -  首先该流量肯定不能超过`c(x, y)`, 因为边有流量限制

    - 对于`x`而言, 其`f[x]`也是由两部分组成:

      1. 流向`y`

         这部分流量的大小为`min(c(x, y), d[y])`

      2.  流向其它相邻节点

         `f[x] - min(c(x, y), d[y])`

    - 由`y`流向`x`的流量, 肯定不能超过由`x`流向其它相邻节点的最大流量, 即`f[x] - min(c(x, y), d[y])`

  递推方程
  
  ![image-20221007091341817](http://www.cdn.liver0377.xyz/typora/202210070913869.png)
  
  当`x`的度数为1的时候,  `x`就只有`y`一个相邻节点, 因此`f[x] - min(c(x, y), d[y])`不用考虑
  
  发现`y`只依赖于父亲节点`x`, 因此可以使用二次扫描 + 换根法



**代码实现**

- 时间复杂度: $O(N)$



```cc
#include <algorithm>
#include <cstring>
#include <iostream>

using namespace std;

typedef long long LL;
const int N = 200010;
const int M = N * 2;

bool v[N];
LL d[N], f[N], deg[N];
LL h[N], e[M], ne[M], w[M], idx;
int T, n;

void add(int a, int b, int c) {
    w[idx] = c, e[idx] = b, ne[idx] = h[a], h[a] = idx++;
    deg[b]++;
}

// 计算出所有的d[i]
void dp(int u) {
    v[u] = true;
    d[u] = 0;
    for (int i = h[u]; ~i; i = ne[i]) {
        int j = e[i];
        if (v[j])
            continue;
        dp(j);
        if (deg[j] == 1)
            d[u] += w[i];
        else
            d[u] += min(d[j], w[i]);
    }
}

// 自顶向下计算出所有的f
void dfs(int u) {
    v[u] = 1;
    for (int i = h[u]; ~i; i = ne[i]) {
        int j = e[i];
        if (v[j])
            continue;
        if (deg[u] == 1)
            f[j] = d[j] + w[i];
        else
            f[j] = d[j] + min(f[u] - min(d[j], w[i]), w[i]);
        dfs(j);
    }
}

int main() {
    cin >> T;
    while (T--) {
        cin >> n;
        memset(v, 0, sizeof v);
        memset(d, 0, sizeof d);
        memset(f, 0, sizeof f);
        memset(h, -1, sizeof h);
        memset(deg, 0, sizeof deg);
        idx = 0;

        for (int i = 1; i <= n - 1; i++) {
            int a, b, c;
            scanf("%d%d%d", &a, &b, &c);
            add(a, b, c);
            add(b, a, c);
        }

        int root = 1;
        dp(root);

        memset(v, 0, sizeof v);
        f[root] = d[root];
        dfs(root);

        LL ans = 0;
        for (int i = 1; i <= n; i++) {
            ans = max(ans, f[i]);
        }
        printf("%lld\n", ans);
    }

    return 0;
}
```

