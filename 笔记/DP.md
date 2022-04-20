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





**题干**

![image-20220417223938970](https://cdn.jsdelivr.net/gh/liver0377/images@main/img/image-20220417223938970.png)

**题解**

- 解法1

  朴素解法

  ![image-20220417224059105](https://cdn.jsdelivr.net/gh/liver0377/images@main/img/image-20220417224059105.png)

  注意这里对集合f[i]的划分:

  - 这里是以最长上升子序列中倒数第二个数的下边进行划分，尽管有些情况是不存在
  - 若a[i] <= a[j], 那么j就不可能作为最长上升子序列的倒数第二个数，因此只在a[i] > a[i]的情况下进行f[i]的更新

  

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
         f[i] = 1;
     }
     
     for (int i = 2; i <= n; i++) {
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

`![image-20220420163548455](https://cdn.jsdelivr.net/gh/liver0377/images@main/img/image-20220420163548455.png)





**解题思路**

![image-20220420163630954](https://cdn.jsdelivr.net/gh/liver0377/images@main/img/image-20220420163630954.png)

- 这里的状态j比较难以理解

  f表示当前列的状态，通过它的二进制位表示可以看出在第i - 1列的哪些行新放置了块

  - 如f = (1011)  // 自右向左看

    那么就表示第i - 1列的第1,2,4行新放置了块

- 这里只需要确定横着的块如何放即可，因为在横着放玩所有的块之后，只需要用竖着的块填充所有的剩余空间即可

- 不可放置情况

  不可放置情况有两个：

  - 情况1：i - 1的第t行新放置一个块，i列的第t行也新放置一个块，此时发生冲突
  - 情况2：任意一列的自上而下的连续空闲区域中，存在偶数个空格，此时竖着的块就放不进来

  当上面两个情况的任意一个情况不满足时，都需要放弃这种状态



**题解**

```cpp
#include <iostream>
#include <algorithm>
#include <cstring>
using namespace std;

const int N = 12, M = 1 << 12; // N 多开1位, M 是表示状态数目的最大值

bool st[M];                    // st[i]表示状态i是否满足情况2
long long f[N][M];
int main() {
    int n, m;
    
    while (cin >> n >> m, n || m) {
        memset(f, 0, sizeof(f));
        memset(st, 0, sizeof(st));
        for (int i = 0; i < 1 << n; i++) {   // 通过这个for循环，我们可以穷举出在:n * m型矩阵中
                                             // 所有可能可行状态(满足情况2)
            st[i] = true;                    // 这里的状态稍微根后面的有点不大一样
                                             // 当第t行不是空格时，那么就将i的第t为就应该为1
            int cnt = 0;                     // 用于对一段连续的空闲区域计数
            for (int j = 0; j < n; j++) {
                
                if (i >> j & 1) {
                   if (cnt % 2) {             // 奇数个空闲块
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
        
       f[0][0] = 1;                     // f[0][0] 表示:
                                        // 从-1列新放置0个块的可能方案，很显然不存在-1列
                                        // 但是可以假象存在这么一个-1列，我们只能在-1列中竖着放置
                                        // 这样的话方案数就是1
       for (int i = 1; i <= m; i++) {   // i代表列数
           for (int j = 0; j < 1 << n; j++) {        // j代表第i列的状态
               for (int k = 0; k < 1 << n; k++) {    // k是用来穷举前一列的状态的
                                                     // 这里通过固定i列的状态j，穷举i - 1列的状态k
                                                     // 找到所有符合条件的前一列状态k, 将这些状态k的方案数累加
                                                     // 就能够得到最终的i列情况下状态为j的方案数
                   if ((j & k) == 0 && st[j | k]) {  // j & k == 0 表示i 列与 i - 1列没有冲突
                                                     
                       f[i][j] += f[i - 1][k];
                   }
               }
           }
       }
       
       cout << f[m][0] << endl;                      // 真正有效的列是0~m - 1列
                                                     // 然而第m - 1列肯定不能新放置块
                                                     // 即假设有m列存在的话，那么m列中伸出来的块肯定为0,
                                                     // 即m列的状态为0
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

