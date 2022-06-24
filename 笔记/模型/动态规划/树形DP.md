### 树的最长路径

![image-20220623162057328](http://www.cdn.liver0377.xyz/typora/202206231620393.png)



**解题思路**

- **路径**： 从树的一个节点走到另一个节点，不能经过重复的边

- 对于一棵给定的树而言，最长路径看起来就像这样
  ![image-20220623162618809](http://www.cdn.liver0377.xyz/typora/202206231626851.png)

- 最长路径可能有多条，对于每条最长路径而言，在树当中，它一定有着一个最高点
  ![image-20220623162738367](http://www.cdn.liver0377.xyz/typora/202206231627409.png)

- 该最高点将路径一分为二，分为两个子路径，并且两个子路径**一定位于不同的子树上**

  > 不然的话就会边就会重复

- 因此只需要对于树中的每个节点，只需要求出经过它并且是向下的**最长路径d1**和**次长路径d2**，且满足这两个路径不为于一棵子树上

  对于每个节点对应的**d1 + d2**, 选出最大的即是最终的答案



**代码实现**

```cpp
#include <iostream>
#include <algorithm>
#include <cstring>

using namespace std;

const int N = 10010;
const int M = N * 2;

int h[N], e[M], w[M], ne[M];
int idx;
int n;
int ans;

void add(int a, int b, int c) {
    e[idx] = b, w[idx] = c, ne[idx] = h[a], h[a] = idx ++;
}

// dfs会返回以u为根节点，经过u的最长路径
int dfs(int u, int father) {
    // 叶子节点的最长路径长度为0
    
    int d1 = 0, d2 = 0; // d1: 对于以u为根节点的树的所有子树，以u为根节点的最长路径
                        // d2: 对于以u为根节点的树的所有子树，以u为根节点的最长路径
                        // 注意，这里的d1, d2不能够在一棵子树上
    for (int i = h[u]; i != -1; i = ne[i]) {
        int j = e[i];
        if (j == father) continue; // 不能够沿着父节点向上走
        int d = dfs(j, u) + w[i];
        if (d >= d1) d2 = d1, d1 = d;
        else if (d > d2) d2 = d;
    }
    
    ans = max(ans, d1 + d2);       // 在dfs过程中，这里的ans会遍历每一个节点
    return d1;
}

int main() {
    cin >> n;
    memset(h, -1, sizeof h);
    
    for (int i = 0; i < n - 1 ;i++) {
        int a, b, c;
        cin >> a >> b >> c ;
        add(a, b, c);
        add(b, a, c);  // 建图
    }    
    
    dfs(1, -1);  // 从任意一个节点进行dfs都可以
    
    cout << ans << endl;
    return 0;
}
```



### 树的中心

![image-20220623172344215](http://www.cdn.liver0377.xyz/typora/202206231723272.png)

**解题思路**

- 对于任意一个节点u来说，其到树中其他节点的最远路径可能有两种情况

  - 该路径向下
    ![image-20220623172815145](http://www.cdn.liver0377.xyz/typora/202206231728187.png)

  - 该路径向上

    - 设`u`的最大路径向上，`u`的父节点为`k`
    - 对于`k`来说，它与其他节点的最大路径可能是向上走，也可能是向下走
    - 这里考虑一种比较特殊的情况：**节点k的最大路径的下一个节点恰好就是节点u**
    -  对于节点`u`来说，它的最大路径一定不能向回走，不然路径就会重复经过边`u - k`
    - 所以，在该情况下的`u`的最大路径当中，第二步只能在**向上走**与**向下次大路径**中选择
    - 向上走
      ![image-20220623174048533](http://www.cdn.liver0377.xyz/typora/202206231740573.png)

    - 向下次大路径

    ![image-20220623174238985](http://www.cdn.liver0377.xyz/typora/202206231742026.png)

  - 算法流程

    - 先采用一遍dfs，求出所有节点的向下走的最大路径以及次大路径
    - 再采用一遍dfs,   对于每个节点, 求出其子节点向上走的最大路径
    - 遍历所有节点向上走以及向下走最大路径，选择最短的

```cc
#include <iostream>
#include <cstring>
#include <algorithm>

using namespace std;

const int N = 10010, M = N << 1, INF = 0x3f3f3f3f;

int n;
int h[N], e[M], w[M], ne[M], idx;
int d1[N], d2[N], up[N];  // d1[i]: i节点向下走的最大路径长度
                          // d2[i]: i节点向下走的次大路径长度
                          // up[i]: i节点向上走的最大路径长度
int s1[N], s2[N];         // s1[i] = k: i节点向下沿着k节点走的路径长度最大
                          // s2[i] = k: i节点向下沿着k节点走的路径长度次大

void add(int a, int b, int c) {
    e[idx] = b, w[idx] = c, ne[idx] = h[a], h[a] = idx ++ ;
}

// 从以u为根节点的子树中向下遍历，求出u节点所有子节点j的d1[j], d2[j]
void dfs1(int u, int father) {
    // d1[u] = d2[u] = -INF;  //这题所有边权都是正的，可以不用初始化为负无穷
    for (int i = h[u]; i != -1; i = ne[i])
    {
        int j = e[i];
        if (j == father) continue;
        dfs1(j, u);           // 先求出u的所有子节点的d1, d2
        if (d1[j] + w[i] >= d1[u])  // 更新d1[u], d2[u]
        {
            d2[u] = d1[u], s2[u] = s1[u]; 
            d1[u] = d1[j] + w[i], s1[u] = j;  // 注意这里要保存s1, s2
        }
        else if (d1[j] + w[i] > d2[u])
        {
            d2[u] = d1[j] + w[i], s2[u] = j;
        }
    }
    // if (d1[u] == -INF) d1[u] = d2[u] = 0; //特判叶子结点
}

// 从u节点开始的子树向下遍历，利用父节点来更新子节点j的up[j]
void dfs2(int u, int father)
{
    // 若u为根节点，那么up[u] = 0
    for (int i = h[u]; i != -1; i = ne[i])
    {
        int j = e[i];
        if (j == father) continue;
        if (s1[u] == j) up[j] = w[i] + max(up[u], d2[u]);   //son_u  = j，则用次大更新
        else up[j] = w[i] + max(up[u], d1[u]);              //son_u != j，则用最大更新
        dfs2(j, u);
    }
}
int main()
{
    memset(h, -1, sizeof h);
    scanf("%d", &n);
    for (int i = 1; i < n; i ++ )
    {
        int a, b, c;
        scanf("%d%d%d", &a, &b, &c);
        add(a, b, c), add(b, a, c);
    }

    dfs1(1, -1);
    dfs2(1, -1);

    int res = INF;
    for (int i = 1; i <= n; i ++ ) res = min(res, max(d1[i], up[i]));
    printf("%d\n", res);

    return 0;
}
```





### 数字转换

![image-20220623194229714](http://www.cdn.liver0377.xyz/typora/202206231942771.png)

![image-20220623194239748](http://www.cdn.liver0377.xyz/typora/202206231942793.png)



**解题思路**

- 题意转化: 在1~n的范围内，任选一个数字，求出其进行转换，且不产生重复数字的最大转换次数

  当数字`i`的约数`s`之和小于`i`时，`i`和`s`就可以相互转化

- 这里的关键思路是将这题转换为一个图

  - 数字`i`仅对应与一个约数和`j`，因此可以将约数和`j`与数字`i`之间建立一条边，表示他们可以相互转化

    当然，这里还需要满足约数和`j`小于`i`本身的条件

  - 将1~n之间的数字全部建图之后，会得到很多树，即**森林**

  - **找最大转换次数其实就是找森林中的最长路径**

  - 因此，只需要对每棵树进行dfs()即可, 方法和`树的最长路径`这题方法是一样的





**代码实现**

```cc
#include <iostream>
#include <algorithm>
#include <string.h>

using namespace std;

const int N = 50010;
const int M = N * 2;

int h[N], e[M], ne[M], w[M], idx;
int sum[N];  // sum[i]: i的约数之和
int st[N];   // st[i] == true: i节点为内部节点或者叶子节点
int n;
int ans;

void add(int a, int b) {
    e[idx] = b, ne[idx] = h[a], h[a] = idx ++;
}

int dfs(int u) {
    int d1 = 0, d2 = 0;
    for (int i = h[u]; i != -1; i = ne[i]) {
        int j = e[i];
        int d = dfs(j);
        if (d + 1 >= d1) {
            d2 = d1;
            d1 = d + 1;
        } else if (d + 1 >= d2) {
            d2 = d + 1;
        }
    }
    
   ans = max(ans, d1 + d2);
   return d1;
}

int main() {
    cin >> n;
    // 1. 算约数和
    for (int i = 1; i <= n; i++) {
        for (int j = 2; j <= n / i; j++) {
            sum[i * j] += i;
        }
    }
    // 2. 建图
    memset(h, -1, sizeof h);
    
    for (int i = 2; i <= n; i++) { // 从2开始，因为1的约数和为0，不在题目范围内部
        if (sum[i] < i) {
            add(sum[i], i);        // 单向边!!!
            st[i] = true;          // i为内部节点或者叶子节点
        }
    }
    
    // 3. dfs
    for (int i = 1; i <= n; i++) {  
        if (!st[i]) {
            dfs(i);
        }
    }
    
    cout << ans << endl;
    return 0;
}
```

- 这里采用算倍数的方式来间接求出了每个数的约数和
- 如果采用试除法，那么对于每个数`i`求它的约数和，需要进行$\sqrt{i}$次计算, 时间复杂度为$O(n\sqrt{n})$
- 采用算倍数的方式时，求每个数`i`的约数和的计算次数为$i + i / 2 + i / 3 + ... = O(nlog(n))$ 





### 二叉苹果树

![image-20220624163705244](http://www.cdn.liver0377.xyz/typora/202206241637321.png)

**解题思路**

- 题意转换： 给定一棵树，树的每一个边都有权值(苹果), 选择一条经过根节点的路径，路径长度限制为m, 求路径最大苹果数目

- 每当选定一个节点，就必须要选择其父节点，因为路径必须经过根节点，所以这题可以转化成一个**有依赖的背包问题**

- 对于以`u`为根节点的树，设其最大能够使用的边的数目为`m`, 即背包容积为`m`

- 对于其每棵子树，就是一个分组，分组的体积为0, 1, 2, 3, ...

  > 注意边数问题，对于子树`son`, 如果其能够使用的最大边数为`m`, 那么`u`分配给子树的边数应该为`m + 1`

**代码实现**

```cc
#include <iostream>
#include <algorithm>
#include <string.h>

using namespace std;

const int N = 110;
const int M = N * 2;

int h[N], ne[M], e[M], w[M], idx;
int f[N][N];
int n, m;

void add(int a, int b, int c) {
    e[idx] = b, w[idx] = c, ne[idx] = h[a], h[a] = idx ++;
}

void dfs(int u, int father) {
    // 叶子节点的苹果树为0
    for (int i = h[u]; i != -1; i = ne[i]) {
        int son = e[i];
        if (son == father) continue;
        dfs(son, u);
        for (int j = m; j >= 0; j --) {
            for (int k = 0; k < j; k++) { // 给子树son的边数为k, k不能取到j
                f[u][j] = max(f[u][j], f[son][k] + f[u][j - k - 1] + w[i]);
            }
        }
    }
}

int main() {
    cin >> n >> m;
    
    memset(h, -1, sizeof h);
    for (int i = 1; i <= n - 1; i++) {
        int a, b, c;
        cin >> a >> b >> c;
        add(a, b, c), add(b, a, c);  // 题干中没有给定父子关系，因此建立无向图
    }
    
     dfs(1, -1);                     // 题干指定根节点为1，只能从1开始遍历
    
     cout << f[1][m] << endl;
     return 0;
}
```



### 战略游戏

![image-20220624174429065](http://www.cdn.liver0377.xyz/typora/202206241744124.png)

![image-20220624174444401](http://www.cdn.liver0377.xyz/typora/202206241744461.png)



**解题思路**

- 此题和**没有上司的舞会**十分类似
- 对于每条边，应该至少有一个士兵位于某个端点



- 状态表示
  ![image-20220624174925182](http://www.cdn.liver0377.xyz/typora/202206241749234.png)
- 状态计算
  - $f[i][0] = \Sigma{f[son_k][1]}$
  - $f[i][1] = \Sigma{min(f[son_k][0], f[son_k][1])}$



**代码实现**

```cc
#include <iostream>
#include <algorithm>
#include <cstring>

using namespace std;

const int N = 1510;
const int M = 1510;

int h[N], e[M], ne[M], idx;
int f[N][2]; // f[u][0]: 不选择根节点u的最小士兵数
             // f[u][1]: 选择根节点u的最小士兵数目
int st[N];
int n;

void add(int a, int b) {
    e[idx] = b, ne[idx] = h[a], h[a] = idx ++;
}

void dfs(int u) {
    f[u][0] = 0;
    f[u][1] = 1;
    
    for (int i = h[u]; i != -1; i = ne[i]) {
        int j = e[i];
        
        dfs(j);
        f[u][0] += f[j][1];
        f[u][1] += min(f[j][0], f[j][1]);
    }
}

int main() {
    while (scanf("%d", &n) == 1) {
        // 1. 初始化
        memset(h, -1, sizeof h);
        memset(e, 0, sizeof e);
        memset(ne, 0, sizeof ne);
        idx = 0;
        memset(f, 0, sizeof f);
        memset(st, 0, sizeof st);
        
        // 2. 建图
        int a, cnt;
        for (int i = 0; i < n; i++) {
            scanf("%d:(%d) ", &a, &cnt);  // 格式化输入
            for (int j = 0; j < cnt; j++) {
                int b;
                cin >> b;
                add(a, b);
                st[b] = true;
            }
        }
        
        int root = 0;  // 节点编号从0开始
        while (st[root]) root ++;
        
        dfs(root);
        
        cout << min(f[root][0], f[root][1]) << endl;
    }
    
    return 0;
}
```





### 皇宫看守

![image-20220624193259980](http://www.cdn.liver0377.xyz/typora/202206241933066.png)



**解题思路**

- 题意转换

  给定一棵树，每个节点有对应的权值，在每个节点上安排守卫，守卫可以观察到与其相连的所有节点

  求保证所有节点都被观察到，并且总守卫权值和最小的经费

- 这题与**战略游戏**很类似，但是这题需要保证点的覆盖，而不是边的覆盖

  > 存在所有点都被覆盖但是所有边没有被覆盖完全的情况，因此这两种覆盖方式不是等价的

- 借助状态机模型，建立三种状态

  - $f[i][0]$: 在以i为根的子树中，根节点不放置，被父节点看到的方案
  - $f[i][1]$: 在以i为根的子树中，根节点不放置，被子节点看到的方案数
  - $f[i][2]$: 在以i为根的子树中，根节点放置，被自己看到的方案数

  > 注意这里状态0与状态1是有**重叠**的，即根节点既可以被父节点看到，也可以被子节点看到
  >
  > 但是在后面的状态转移过程中，并没有发生$f[i][0]$ + $f[i][1]$的情况，因此
  >
  > 这种重叠并不会影响最终的答案

- 状态转移方程

  - $f[i][0]$

    如果根节点`i`位于状态0，那么其所有的子节点`j`可以处于状态1或者状态2，但是不能处于状态0

    $f[i][0] = \Sigma_{son} {min(f[son][1], f[son][2])}$

  - $f[i][1]$

    如果根节点`i`位于状态1，那么其所有子节点要么位于状态1，要么位于状态2，并且一定有**一个或多个**子节点一定位于状态2

    $f[i][1] = min(f[son][2] + \Sigma_{son} {min(f[other\_son][1], f[other\_son][2])})$

    这里的$other\_son$表示除了当前的子节点`son`之外的其它所有节点

  - $f[i][2]$

    如果根节点`i`位于状态2, 那么其子节点可以处于任意一种状态

    $f[i][2] = min(f[son][0], f[son][1], f[son][2]) + w[son]$

     

**代码实现**

```cc
#include <iostream>
#include <algorithm>
#include <cstring>

using namespace std;

const int N = 1510;
const int M = 1510;

int h[N], e[M], ne[M], idx;
int f[N][3];  // f[i][0]: 在以i为根的子树中，根节点不放置，被父节点看到的方案数
              // f[i][1]: 在以i为根的子树中，根节点不放置，被子节点看到的方案数
              // f[i][2]: 在以i为根的子树中，根节点放置，被自己看到的方案数
              
int w[N];     // w[i]: 点i的权值
bool st[N];    // st[i]: 点i是否为内部节点或者叶子节点
int n;

void add(int a, int b) {
    e[idx] = b, ne[idx] = h[a], h[a] = idx ++;
}

void dfs(int u) {
    f[u][0] = 0;
    f[u][1] = 0x3f3f3f3f;
    f[u][2] = w[u];
    
    for (int i = h[u]; i != -1; i = ne[i]) {
        int j = e[i];
        dfs(j);
        f[u][0] += min(f[j][1], f[j][2]);
        f[u][2] += min({f[j][0], f[j][1], f[j][2]});
    }
    
    for (int i = h[u]; i != -1; i = ne[i]) {
        int j = e[i];
        // 这里采用了间接方式, f[u][0] - min(f[j][1], f[j][2]) 即代表其它所有节点
        f[u][1] = min(f[u][1], f[j][2] + f[u][0] - min(f[j][1], f[j][2]));
         
    }
}

int main() {
    cin >> n;
    
    memset(h, -1, sizeof h);
    for (int i = 0; i < n; i++) {
        int id, cnt;
        cin >> id >> w[id] >> cnt;
        for (int j = 0; j < cnt; j++) {
            int a;
            cin >> a;
            add(id, a);
            st[a] = true;
        }
    }
    
    int root = 1;
    while (st[root]) root ++;
    
    dfs(root);
    
    cout << min(f[root][1], f[root][2]) << endl;
    return 0;
}
```

