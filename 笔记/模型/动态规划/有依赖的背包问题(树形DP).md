**题干**

![image-20220528101717025](http://www.cdn.liver0377.xyz/typora/202205281017103.png)

![image-20220528101728688](http://www.cdn.liver0377.xyz/typora/202205281017734.png)

![image-20220528101741626](http://www.cdn.liver0377.xyz/typora/202205281017658.png)





**解题思路**

对于以`u`为根节点的树, 可以根据`u`给子树分配的体积进行划分

- 假设根节点可以使用的体积为j, 那么对于剩下的所有子节点而言，他们能够使用的最大体积就是j - v[i]
- **将每个子节点看做是一个物品体积为0, 1, 2,...,j - v[i]的分组**
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

// 
// 从以u为根的节点开始遍历，计算出所有的f[i][j], 其中1 <= i <= n, 1 <= j <= m
void dfs(int u) {
    // 1. 计算出所有的f[u][x], 这里的f[u][x]表示不选择u节点，总可用体积为x的, 0 <= x <= m - v[u] 
    for (int i = h[u]; i != -1; i = ne[i]) {  
        int son = e[i];
        dfs(son);                             // 需要首先计算出f[son][...]  
        for (int j = m - v[u]; j >= 0; j --) {// 当遍历到以u为根的子树时，可用的体积并不知道，这里
                                              // 最大可能的体积为m - v[u]
            for (int k = 0; k <= j; k++) {    // 子树son中花费的体积为k
                
                f[u][j] = max(f[u][j], f[u][j - k] + f[son][k]); // 这里的f[u][j - k]其实还可以继续从子树son                                                                  // 中分配体积，但是这不影响f[u][j]的计算
            }
        }
    }
    
    // 2. 更新f[u][j]
    //  上一步计算出的f[u][j]是不包含根节点u本身, 体积为j的选法的最大价值
    //  现在统一修改
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

  
