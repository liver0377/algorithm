**题干**

![image-20220528101717025](http://www.cdn.liver0377.xyz/typora/202205281017103.png)

![image-20220528101728688](http://www.cdn.liver0377.xyz/typora/202205281017734.png)

![image-20220528101741626](http://www.cdn.liver0377.xyz/typora/202205281017658.png)





**解题思路**

- 假设根节点可以使用的体积为`j`, 那么对于剩下的所有子节点而言，他们能够使用的最大体积就是`j - v[i]`

- **将每个子树看做是一个分组**, 在子树中的任何一种选择方案代表着分组中的一个物品

  假设该方案(物品)的体积为`k`, 子树根节点为`son`, 那么该方案(物品)的价值就是`f[son][k]`

  实际上, 由于并不清楚从子树中选到底有多少种方案, 索性**直接将子树看做是有j个物品, j为为该分组分配的体积大小**

- 接下来就是一个分组背包问题了, 每颗子树(分组)可以选或者不选, 如果选的话,最多只能选择一件物品(方案)

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
int f[N][N];   // f[i][j]: 在以u为根的子树中, 选择根节点, 并且总体体积不超过j的最大价值
int w[N], v[N];
int root;
int n, m;

// 使用邻接表来存储所有节点
void add(int a, int b) {
    e[idx] = b, ne[idx] = h[a], h[a] = idx ++;
}


// 计算出所有的f[u][k], k = 0, 1, ..., m
void dfs(int u) {
    // 1. 计算出所有的f[u][x], 这里的f[u][x]表示不选择u节点，总可用体积为x的, 0 <= x <= m - v[u] 
    for (int i = h[u]; i != -1; i = ne[i]) {  
        int son = e[i];
        dfs(son);   // 经过dfs(son)之后, 所有的f[son][k]表示的是在son子树中, 选择son, 体积为k的最大价值 
        for (int j = m - v[u]; j >= 0; j --) { // 0 ~ m - v[u]为步骤2所需要的范围
            for (int k = 0; k <= j; k++) {    // 子树son中花费的体积为k(选择哪个物品)
                
                f[u][j] = max(f[u][j], f[u][j - k] + f[son][k]); // f[son][k]表示从son子树中选择体积为k的物品                                                                  // 接下来继续在整棵以u为根的子树中使用j - k来                                                                  // 选, 因为这里不像普通分组背包, 无法选前一组                                                                  // 选
            }
        }
    }
    
    // 2. 更新f[u][j]
    //  上一步计算出的f[u][j]是不包含根节点u本身, 体积为j的选法的最大价值
    //  然而实际上u必须要选, 之前的操作只是对当前这一步的预处理, 计算出前置的f[u][k]
    for (int j = m; j >= v[u]; j--) f[u][j] = f[u][j - v[u]] + w[u];
    
    // 3. 修正非法情况
    // 当 j < v[u]时, 背包不能够容纳根节点
    // 在1过程中，f[u][j]是合法的，但在2过程中, f[u][j]的含义发生了变化，因此需要修正
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



- 注意点

  在有依赖的背包问题中，存在着下面的范式

  ```
  for 枚举物品组
    for 枚举体积    // 必须从大到小来枚举
       for 枚举每个分组内部的决策
      
  ```

  
