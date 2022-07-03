### 池塘计数

![image-20220703161434805](http://www.cdn.liver0377.xyz/typora/202207031614874.png)



**解题思路**

- 此题就是要求连通块的数目，这里的连通是8连通，采用`bfs`来做

- 算法流程

  - 从上之下，从左至右遍历每一个点
  - 如果该点未被访问过且是水的话，就以该点为起点，采用`bfs`将其所在的连通块的所有点全部标记，ans++
  - `bfs`结束只有，继续遍历后面的点，最终，当整个矩阵全部被遍历后，返回`ans`

- 如何将一个点所在的连通块全部标记？

  即基本`bfs`，维护一个队列，首先将起点放入队列

  接下来每次从队列头部取出一个点，这里由于是8连通，因此需要访问该点周围的8个点

  将8个点当中任何一个与当前点连通且合法的点加入队列，并且将其标记

  当队列为空时，整个联通块也就全部标记完毕



**代码实现**

```cc
#include <iostream>
#include <algorithm>

using namespace std;

#define x first
#define y second

const int N = 1e3 + 10;
const int M = N * N;
typedef pair<int, int> PII;

PII q[M];
char g[N][N];
bool st[N][N];
int n, m;


void bfs(int sx, int sy) {
    int hh = 0, tt = 0;
    q[0] = {sx, sy};
    st[sx][sy] = true;
    
    while (hh <= tt) {
        PII t = q[hh ++];
        
        for (int i = t.x - 1; i <= t.x + 1; i++) {
            for (int j = t.y - 1; j <= t.y + 1; j++) {
                if (i == t.x && j == t.y) continue;
                if (g[i][j] == '.' || st[i][j]) continue;
                if (i < 0 || j < 0 || i >= n || j >= m) continue;
                
                q[++ tt] = {i, j};
                st[i][j] = true;
            }
        }
    }
}

int main() {
    cin >> n >> m;
    for (int i = 0; i < n; i++) scanf("%s", g[i]);
    
    int cnt = 0;
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < m; j++) {
            if (g[i][j] == 'W' && !st[i][j]) {
                bfs(i, j);
                cnt ++;
            }
        }
    }
    
    cout << cnt << endl;
    
    return 0;
}
```





### 城堡问题

![image-20220703165533713](http://www.cdn.liver0377.xyz/typora/202207031655773.png)

![image-20220703165548053](http://www.cdn.liver0377.xyz/typora/202207031655104.png)



**解题思路**

- 这题的输入输出比较繁琐，可以采用二进制的思路来处理

  ![image-20220703165839980](http://www.cdn.liver0377.xyz/typora/202207031658032.png)

- 通过获取方格对应数字上各位的二进制位，就可以直到该方格与其他方格之间是否有墙

- `bfs`的思路还是是上一题一样，不过这里是4连通

- 对于4连通，通常会使用`dx`,`dy`数组，然后采用一个`for`来遍历周围的四个点

  ```cc
  const int ddx[4] = {0, -1, 0, 1}, y[4] = {-1, 0, 1, 0};
  for (int i = 0; i < 4; i++) {
      int nx = x + dx[i], ny = y + dy[i];
      ...
  }
  ```

- 关于连通块的面积，只需要在入队或者出队时进行计数操作即可，入队/出队的次数即为连通块的面积

**代码实现**

```cc
#include <iostream>
#include <algorithm>
#include <cstring>

using namespace std;

#define x first
#define y second

typedef pair<int, int> PII;

const int N = 55;
int g[N][N];
PII q[N * N];
bool st[N][N];
int n, m;

// 返回连通块的面积
int bfs(int sx, int sy) {
  const int dy[4] = {-1, 0, 1, 0}, dx[4] = {0, -1, 0, 1};
  int hh = 0, tt = 0;
 
  q[0] = {sx, sy};
  st[sx][sy] = true;
  
  int area = 0; 
  while (hh <= tt) {
      area ++;
      PII t = q[hh ++];
      for (int i = 0; i < 4; i++) {
          int nx = t.x + dx[i], ny = t.y + dy[i];
          if (nx < 0 || nx >= n || ny < 0 || ny >= m) continue;
          if (st[nx][ny]) continue;
          if (g[t.x][t.y] >> i & 1) continue;  // 判断点t与该点之间是否有墙
          
          q[++ tt] = {nx, ny};
          st[nx][ny] = true;
      }
  }
  
  return area;
}

int main() {
    cin >> n >> m;
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < m; j++) {
            cin >> g[i][j];
        }
    }
    
    int area = 0, cnt = 0;
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < m; j++) {
            if (!st[i][j]) {
                area = max(area, bfs(i, j));
                ++ cnt;
            }
        }
    }
    
    cout << cnt << endl;
    cout << area << endl;
    
    return 0;
}
```



### 山峰与山谷

![image-20220703173425079](http://www.cdn.liver0377.xyz/typora/202207031734146.png)



**解题思路**

- 这题与周围两题的区别在于需要判断周围格子与当前格子的关系

- 如果周围格子高于当前格子，说明存在其他连通块高于当前连通块

  如果周围格子低于当前格子，说明存在其他连通块低于当前连通块

- 如果一个连通块周围没有连通块比它高，那么它是山峰

  如果一个连通块周围没有连通块比它低，那么它是山谷

  一个连通块中的所有格子的高度全部相等，并且既可以是山峰，也可以是山谷

- 这题是一个8连通

**代码实现**

```cc
#include <iostream>
#include <algorithm>

using namespace std;

#define x first
#define y second
typedef pair<int, int> PII;

const int N = 1e3 + 10;
const int M = N * N;

int h[N][N];
bool st[N][N];
PII q[M];
int n;

void bfs(int sx, int sy, bool& has_higher, bool& has_lower) {
   int hh = 0, tt = 0;
   q[0] = {sx, sy};
   
   while (hh <= tt) {
       PII t = q[hh++];
       
       for (int i = t.x - 1; i <= t.x + 1; i++) {
           for (int j = t.y - 1; j <= t.y + 1; j++) {
               if (i < 0 || j < 0 || i >= n || j >= n) continue;
               if (h[t.x][t.y] != h[i][j]) {  // 发现了其他连通块，将其和当前连通块进行比较
                   if (h[i][j] > h[t.x][t.y]) 
                       has_higher = true;
                    else 
                       has_lower = true;
               } else if (!st[i][j]) {
                   q[++ tt] = {i, j};
                   st[i][j] = true;
               }
           }
       }
   }
}

int main() {
    cin >> n;
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < n; j++) {
            cin >> h[i][j];
        }
    }
    
    int peak = 0, valley = 0;
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < n; j++) {
            bool has_higher = false, has_lower = false;
            if (!st[i][j]) {
                bfs(i, j, has_higher, has_lower);
                if (!has_higher) peak++;
                if (!has_lower) valley++;
            }
        }
    }
    
    cout << peak << " " << valley << endl;
    
    return 0;
}
```

