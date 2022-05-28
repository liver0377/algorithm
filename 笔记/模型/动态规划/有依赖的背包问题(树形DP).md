**题干**

![image-20220528101717025](http://www.cdn.liver0377.xyz/typora/202205281017103.png)

![image-20220528101728688](http://www.cdn.liver0377.xyz/typora/202205281017734.png)

![image-20220528101741626](http://www.cdn.liver0377.xyz/typora/202205281017658.png)





**解题思路**

这是一道树形DP的问题，以往 线性背包DP 的 状态转移，第 i 件 物品 只会依赖第 i−1 件 物品 的状态

如果本题我们也采用该种 状态依赖关系 的话，对于节点 i，我们需要枚举他所有子节点的组合 $2^k$ 种可能

再枚举 体积，最坏时间复杂度 可能会达到 $O(N×2^N×V)$（所有子节点都依赖根节点）

因此我们需要换一种思考方式，那就是**枚举每个状态分给各个子节点的体积**

- 假设根节点可以使用的体积为j, 那么对于剩下的所有子节点而言，他们能够使用的最大体积就是j - v[i]
- **将每个子节点看做是一个物品体积为0, 1, 2,...,m的包含m + 1个物品的分组**
- 这样就可以将每个子节点转化为一个分组，整个树的状态计算就可以转化为一个分组背包的计算问题

-  时间复杂度：O(N×V×V)

![image-20220528103515816](http://www.cdn.liver0377.xyz/typora/202205281035894.png)



**题解**

```cpp
#include <iostream>
#include <algorithm>
#include <cstring>

using namespace std;

const int N = 110;

int e[N], ne[N], h[N], idx;
int f[N][N];
int w[N], v[N];
int root;
int n, m;

// 使用邻接表来存储所有节点
void add(int a, int b) {
    e[idx] = b, ne[idx] = h[a], h[a] = idx ++;
}

// 采用dfs遍历每个节点,将每棵树看成是一个分组背包,每个子节点就是一个分组
// 计算根节点编号为u的状态
void dfs(int u) {
    for (int i = h[u]; i != -1; i = ne[i]) {  // 遍历所有子节点----循环分组
        int son = e[i];
        dfs(son);                             // 计算所有的f[son][...]  
        for (int j = m - v[u]; j >= 0; j --) {// 遍历所有可能的体积-循环体积
            for (int k = 0; k <= j; k++) {    // 每个物品的体积为k---循环决策
                f[u][j] = max(f[u][j], f[u][j - k] + f[son][k]);
            }
        }
    }
    
    // 之前计算出的f[u][...]均未包含根节点自身, 这里统一加上
    for (int j = m; j >= v[u]; j--) f[u][j] = f[u][j - v[u]] + w[u];
    // 在前面的循环中，f[u][j], j < v[u]的值也被计算出了，但由于必须得选根节点，因此之前计算出的j<v[u]的状态是不合法的
    // 这里将它们统一置为0
    for (int j = 0; j < v[u]; j++) f[u][j] = 0;
}

int main() {

    cin >> n >> m;
    
    memset(h, -1, sizeof h);
    
    for (int i = 1; i <= n; i++) {
        int  p;
        cin >> v[i] >> w[i] >> p;
        if (p != -1) {
            add(p, i);
        } else {
            root = i;
        }
    }
    
    dfs(root);
    cout << f[root][m] << endl;
    return 0;
}
```



