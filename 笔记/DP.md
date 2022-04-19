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

