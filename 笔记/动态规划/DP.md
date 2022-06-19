### 线性DP

**题干**

![image-20220417223754665](https://cdn.jsdelivr.net/gh/liver0377/images@main/img/image-20220417223754665.png)



**题解**

![image-20220417223907654](https://cdn.jsdelivr.net/gh/liver0377/images@main/img/image-20220417223907654.png)

```cpp
#include <iostream>
#include <algorithm>

using namespace std;

const int N = 510;
const int INF = 1e9;

int a[N][N];
int f[N][N];
int n;

int main() {
    cin >> n;
    
    for (int i = 1; i <= n; i++) {
        for (int j = 1; j <= i; j++) {
            cin >> a[i][j];
        }
    }
    for (int i = 0; i <= n; i++) {
        for (int j = 0; j <= i + 1; j ++) {
            f[i][j] = -INF;
        }
    }
    
    f[1][1] = a[1][1];
    for (int i = 2; i <= n; i++) {
        for (int j = 1; j <= i; j++) {
            f[i][j] = max(f[i - 1][j - 1], f[i - 1][j]) + a[i][j];
        }
    }
    
    int m = -INF;
    for (int i = 1; i <= n; i++) m = max(m, f[n][i]);
    
    cout << m << endl;
    return 0;
}
```



**相似题目**

- [acwing1017](https://www.acwing.com/problem/content/1019/)



**题干**

![image-20220417223938970](https://cdn.jsdelivr.net/gh/liver0377/images@main/img/image-20220417223938970.png)

**题解**

- 解法1

  朴素解法

  ![image-20220417224059105](https://cdn.jsdelivr.net/gh/liver0377/images@main/img/image-20220417224059105.png)

  注意这里对集合f[i]的划分:

  - 这里是以最长上升子序列中倒数第二个数的下边进行划分，尽管有些情况是不存在
  - 若a[i] <= a[j], 那么j就不可能作为最长上升子序列的倒数第二个数，因此只在a[i] > a[j]的情况下进行f[i]的更新

  

  ```cpp
  #include <iostream>
  #include <algorithm>
  
  using namespace std;
  
  const int N = 1010;
  int n;
  int a[N], f[N];
  int main() {
     cin >> n;
     
     for (int i = 1; i <= n; i++) {
         cin >> a[i];
     }
     
     for (int i = 1; i <= n; i++) {
         f[i] = 1;
         for (int j = 1; j < i; j++) {
             if (a[i] > a[j]) {
                 f[i] = max(f[i], f[j] + 1);
             }
         }
     }
     
     int res = 0;
     for (int i = 1; i <= n; i++) res = max(res, f[i]);
     
     cout << res;
     
     return 0;
  }
  ```

  







### 区间DP

**题干**

![image-20220419222542216](https://cdn.jsdelivr.net/gh/liver0377/images@main/img/image-20220419222542216.png)



**基本思路**

![image-20220419222654648](https://cdn.jsdelivr.net/gh/liver0377/images@main/img/image-20220419222654648.png)

**题解**



```cpp
#include <iostream>
#include <algorithm>

using namespace std;

const int N = 310;
int n;
int a[N], s[N], f[N][N];

int main() {
    cin >> n;
    
    for (int i = 1; i <= n; i++) {
        cin >> a[i];
        s[i] = s[i - 1] + a[i];
    }
    
    for (int len = 2; len <= n; len ++) {         // 枚举len
        for (int i = 1; i + len - 1 <= n; i++) {
            int l = i, r = i + len - 1;
            f[l][r] = 1e8;                        // 初始化为一个较大值，以避免后面f[l][r]一直为0
            for (int k = l; k < r; k++) {
                f[l][r] = min(f[l][r], f[l][k] + f[k + 1][r] + s[r] - s[l - 1]);
            }
        }
    }
    cout << f[1][n] << endl;
    return 0;
}
```

- 这里的s是前缀和，因为不管怎样进行合并i\~j区间内的石子，最后一步的代价一定是区间i\~j石子的总质量之和





### 状态压缩DP

**题干**

![image-20220420163527707](https://cdn.jsdelivr.net/gh/liver0377/images@main/img/image-20220420163527707.png)

![image-20220529224240106](http://www.cdn.liver0377.xyz/typora/202205292242163.png)





**解题思路**

![image-20220420163630954](https://cdn.jsdelivr.net/gh/liver0377/images@main/img/image-20220420163630954.png)

- 这里的状态j比较难以理解

  f表示当前列的状态，通过它的二进制位表示可以看出在第i - 1列的哪些行新放置了块

  - 如f = (1011)  // 自右向左看

    那么就表示第i - 1列的第1,2,4行新放置了块

- 这里只需要确定横着的块如何放即可，因为在横着放玩所有的块之后，只需要用竖着的块填充所有的剩余空间即可

- 不合法情况

  不合法情况有两个：

  - 情况1：i - 1列的第t行新放置一个块，i列的第t行也新放置一个块，此时发生冲突
  
  - 情况2：前一列的自上而下的连续空闲区域中，存在奇数个空格
  
    - 这种情况表明前一列一定还可以再插入一个横着的块，由于题干中询问的是将方格装满的方案数，
  
      如果留下奇数个空格，那么后续在放入竖着的块时，一定会导致方格不会被填满，因此方案无效
  
  当上面两个情况的任意一个情况不满足时，都需要放弃这种状态



**题解**

```cpp
#include <iostream>
#include <algorithm>
#include <cstring>
using namespace std;

const int N = 12, M = 1 << 12; // N 多开1位, M 是表示状态数目的最大值，这里也多开一位

bool st[M];                    // st[i]表示状态i是否满足情况2
                               // 该数组用于对状态进行预处理
long long f[N][M];
int main() {
    int n, m;
    
    while (cin >> n >> m, n || m) {
        memset(f, 0, sizeof(f));
        memset(st, 0, sizeof(st));
        for (int i = 0; i < 1 << n; i++) {   // 枚举一列的所有可能状态
                                      
            st[i] = true;                    // 这里的状态稍微跟后面的有点不大一样
                                             // 当第t行不是空格时，那么i的第t位就应该为1
            
            int cnt = 0;                     // 用于对一段连续的空闲区域计数
            for (int j = 0; j < n; j++) {    // 遍历状态的每一位，判断连续空闲区域空格是否为奇数
                
                if (i >> j & 1) {             
                   if (cnt % 2) {             // 奇数个空闲块，则表明前一列状态不合法
                       st[i] = false;
                       cnt = 0;
                       break;
                   }
                } else {
                    cnt ++;
                }
            }
            if (cnt % 2) {
                st[i] = false;
            }
        }   
        
       f[0][0] = 1;                     // f[0][0]: 第0列其实+不存在，可以当做是在第0列一个新的横块都不放, 方案数为1 
                                    
       for (int i = 1; i <= m; i++) {   // i代表列数
           for (int j = 0; j < 1 << n; j++) {        // j代表第i列的状态
               for (int k = 0; k < 1 << n; k++) {    // k是用来穷举前一列的状态的
                                                     // 这里通过固定i列的状态j，穷举i - 1列的状态k
                                                     // 找到所有符合条件的前一列状态k, 将这些状态k的方案数累加
                                                     // 就能够得到最终的i列情况下状态为j的方案数
                   
                   // 没有冲突且叠加后的当前状态合法
                   if ((j & k) == 0 && st[j | k]) {  // j & k == 0 表示i 列与 i - 1列没有冲突
                                                     // 由于第i - 1列会伸出一个格子到第i列，j | k 就能够表示出
                                                     // 两个状态叠加之后第i列的状态
                                                     // 若j | k == (1111)2,即表明第i列的每一行都被占满了
                                                     
                       f[i][j] += f[i - 1][k];       // 当前第i列的状态为j, 前一列状态为k
                   }
               }
           }
       }
       
       cout << f[m][0] << endl;                      // 最终第m行肯定不能放置新块,所以是f[m][0]
    }
    
    return 0;
}
```





**题干**

![image-20220420174445817](https://cdn.jsdelivr.net/gh/liver0377/images@main/img/image-20220420174445817.png)



**题解**

依旧是采用状态压缩

![image-20220420174523889](https://cdn.jsdelivr.net/gh/liver0377/images@main/img/image-20220420174523889.png)

```cpp
#include <iostream>
#include <algorithm>
#include <cstring>

using namespace std;

const int N = 21, M = 1 << 21;
int f[M][N];
int w[N][N];

int main() {
    int n;
    cin >> n;
    
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < n; j++) {
            cin >> w[i][j];
        }
    }
    
    memset(f, 0x3f, sizeof (f));
    f[1][0] = 0; 
    for (int i = 0; i < 1 << n; i++) {
        for (int j = 0; j < n; j++) {
            if ((i >> j) & 1) {         // i所表示的状态应该能够到达j
                for (int k = 0; k < n; k++) {
                    if (j != k) {    // 去除j点，穷举其它所有点的情况, 这里的k代表倒数第二个点
                        f[i][j] = min(f[i][j], f[i - (1 << j)][k] + w[k][j]);
                    }
                }
            }
        }
    }
    
    cout << f[(1 << n) - 1][n - 1] << endl;  // (1 << n) - 1代表的状态是经过所有点
    return 0;
}
```





### 空间优化

DP问题通常有着两种空间优化策略：**双变量**以及**滚动数组**

**双变量**

在计算斐波那契数列时，可以使用动态规划来做

记dp[n]为第n项斐波那契数,则很显然有: dp[n] = dp[n - 1] + dp[n - 2\](n > 2)

这里可以不用一个数组来存储所有的状态，因为当计算第i个值时，只需要直到前两个状态即可，因此可以只使用三个变量来维护这个状态

**例题**

- [leetcode509](斐波那契数)





### 树形DP

**题干**

![image-20220506210437074](https://cdn.jsdelivr.net/gh/liver0377/images@main/img/image-20220506210437074.png)



**题解**

![image-20220506210528671](https://cdn.jsdelivr.net/gh/liver0377/images@main/img/image-20220506210528671.png)

- 树形DP理解起来比较简单，因为主要是使用递归



**核心代码**

```cpp
int n;
int h[N], e[N], ne[N], idx;     
int happy[N];
int f[N][2];
bool has_fa[N];         // 标记是否存在父节点

void add(int a, int b)
{
    e[idx] = b, ne[idx] = h[a], h[a] = idx ++ ;
}

void dfs(int u)
{
    f[u][1] = happy[u];     // 选取根节点的初值为自身的幸福度

    // 遍历子树
    for (int i = h[u]; ~i; i = ne[i])
    {
        int j = e[i];       // 子结点
        dfs(j);             // 递归子结点

        // 状态转移
        f[u][1] += f[j][0]; 
        f[u][0] += max(f[j][0], f[j][1]);
    }
}

// 核心
memset(h, -1, sizeof h);            // 初始化邻接表头指针

// 读入树结构
for (int i = 0; i < n - 1; i ++ )
{
    int a, b;
    scanf("%d%d", &a, &b);
    add(b, a);                      // 尽管是无向图，但只需要保留一条边（上司指向下属）
    has_fa[a] = true;               // 标记存在父节点
}

// 找树根，不存在父节点的就是树根
int root = 1;
while (has_fa[root]) root ++ ;

dfs(root);      // 从根节点开始遍历

printf("%d\n", max(f[root][0], f[root][1]));
```

