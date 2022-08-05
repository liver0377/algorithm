- `bfs`判断最短路模型的条件是点之间的边权为1



### 走迷宫

![image-20220704152747309](http://www.cdn.liver0377.xyz/typora/202207041527400.png)



**解题思路**

- 使用一个数组$d[i][j]$来表示到达点(i, j)的最短距离

- 使用`bfs`来做，起点为(0, 0)

- 如果存在一条从(0, 0)到右下角的通路，那么就表明(0, 0)和右下角的格子位于一个连通块当中

- 当使用`bfs`时，第一次搜索到点(i, j)时，距离是最短的，可以使用一个递推式来计算出到达点(i, j)的距离

  $d[i][j] = d[x][y] + 1$, 从(x, y)可以第一次搜索到点(i, j) 



**代码实现**

- 手写队列

```cc
#include <iostream>
#include <algorithm>

using namespace std;

const int N = 110;
const int M = N * N;
#define x first
#define y second
typedef pair<int, int> PII;

bool st[N][N];
int g[N][N];
int d[N][N];
PII q[M];
int n, m;

int bfs() {
    int hh = 0, tt = 0;
    q[0] = {0, 0};
    st[0][0] = true;
    
    const int dx[4] = {0, -1, 0, 1}, dy[4] = {-1, 0, 1, 0};
    while (hh <= tt) {
       auto t = q[hh ++];
       for (int i = 0; i < 4; i++) {
           int nx = t.x + dx[i];
           int ny = t.y + dy[i];
           if (nx < 0 || ny < 0 || nx >= n || ny >= m) continue;
           if (st[nx][ny]) continue;
           if (g[nx][ny]) continue;
           
           d[nx][ny] = d[t.x][t.y] + 1;
           q[++ tt] = {nx, ny};
           st[nx][ny] = true;
       }
    }
    return d[n - 1][m - 1];
}

int main() {
    cin >> n >> m;
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < m; j++){
            cin >> g[i][j];
        }
    }
    
    cout << bfs() << endl;
}
```



### 迷宫问题

![image-20220704160038648](http://www.cdn.liver0377.xyz/typora/202207041600708.png)



**解题思路**

- 此题相对于第一题走迷宫而言，只是多了一个输出路径而已
- 具体的解法就是将`st`数组升级为`PII pre[i][j]`， 记录(i, j)点的路径上的前一个点
- 给予`pre[i][j]`一个默认值-1，如果`pre[i][j]`的值被修改，那么就表明它被访问过了
- 最后从(n - 1, n - 1)沿着`pre`即可得到路径

**代码实现**

```cc
#include <iostream>
#include <algorithm>
#include <cstring>

#define x first
#define y second

using namespace std;

typedef pair<int, int> PII;

const int N = 1e3 + 10;
const int M = N * N;

int g[N][N];
PII pre[N][N];
PII q[M];
int n;

void bfs(int sx, int sy) {
    int hh = 0, tt = 0;
    q[0] = {sx, sy};
    memset(pre, -1, sizeof pre);
    pre[sx][sy] = {0, 0};  // 非[-1, -1]即可
    
    const int dx[4] = {0, -1, 0, 1}, dy[4] = {-1, 0, 1, 0};
    while (hh <= tt) {
        PII t = q[hh ++];
        for (int i = 0; i < 4; i++) {
            int nx = t.x + dx[i];
            int ny = t.y + dy[i];
            if (nx < 0 || ny < 0 || nx >= n || ny >= n) continue;
            if (pre[nx][ny].x != -1 && pre[nx][ny].y != -1) continue;
            if (g[nx][ny]) continue;
            
            q[++ tt] = {nx, ny};
            pre[nx][ny] = {t.x, t.y};
        }
    }
}

int main() {
    cin >> n;
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < n; j++) {
            cin >> g[i][j];
        }
    }
    
    bfs(n - 1, n - 1);
    
    PII end(0, 0);
    while (true) {
        cout << end.x << " " << end.y << endl;
        if (end.x == n - 1 && end.y == n - 1) break;
        end = pre[end.x][end.y];
    }
    
    return 0;
}
```

- 这里为了能够输出从(0, 0)到(n - 1, n - 1)的路径，直接反向从(n - 1, n - 1)开始进行`bfs`, 这样就能够直接使用`pre`数组正向输出路径了







### 武士风度的牛

![image-20220704163509833](http://www.cdn.liver0377.xyz/typora/202207041635881.png)

![image-20220704163532418](http://www.cdn.liver0377.xyz/typora/202207041635464.png)

- 这里牛的走法就是中国象棋中的马的走法， 如果能够从一个点走日跳到另一个点，那么这两个点就是连通的
- 从一个点可以跳到周围8个点，因此每从队列总取出一个点，就要尝试搜索从该点可以跳到的周围8个点
- 其他做法就是普通`bfs`了



**代码实现**

```cc
#include <iostream>
#include <algorithm>
#include <cstring>

#define x first
#define y second
using namespace std;

const int N = 155;
const int M = N * N;
typedef pair<int, int> PII;

char g[N][N];
int dist[N][N];  // dist[i][j]: 到达点(i, j)的最短跳数
PII q[M];
int n, m;

// 返回到达目的地的跳数(距离)
int bfs() {
    int sx, sy;
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < m; j++) {
            if (g[i][j] == 'K') {
                sx = i, sy = j;
                break;
            } 
        }
    }
    
    int hh = 0, tt = 0;
    q[0] = {sx, sy};
    memset(dist, -1, sizeof dist);
    dist[sx][sy] = 0;
    while (hh <= tt) {
        PII t = q[hh ++];
        const int dx[8] = {-2, -1, 1, 2, 2, 1, -1, -2};
        const int dy[8] = {1, 2, 2, 1, -1, -2, -2, -1};
        
        for (int i = 0; i < 8; i++) {
            int nx = t.x + dx[i];
            int ny = t.y + dy[i];
            if (nx < 0 || ny < 0 || nx >= n || ny >= m) continue;
            if (dist[nx][ny] != -1) continue;  // 访问过
            if (g[nx][ny] == '*') continue;
            
            if (g[nx][ny] == 'H') return dist[t.x][t.y] + 1;  // 搜索到目的地
            dist[nx][ny] = dist[t.x][t.y] + 1;
            q[++ tt] = {nx, ny};
        }
        
    }
    return -1;
}

int main() {
    cin >> m >> n;
    for (int i = 0; i < n; i++) cin >> g[i];
    
    cout << bfs() << endl;
    
    return 0;
}

```





### 抓住那头牛

![image-20220704171711008](http://www.cdn.liver0377.xyz/typora/202207041717069.png)



**解题思路**

- 此题并没有一个二维的矩阵，需要自己建图
- 如果可以从一个点跳到另一个点，那么这两个点之间就可以连一条边
- 如果是第一次连边，那么距离就是最短的，这是`bfs`的性质决定的



**代码实现**

```cc
#include <iostream>
#include <algorithm>
#include <cstring>

using namespace std;

const int N = 1e5 + 10;
int dist[N];
int n, k;
int q[N];

int bfs() {
    int hh = 0, tt = 0;
    q[0] = n;
    
    memset(dist, -1, sizeof dist);
    dist[n] = 0;
    while (hh <= tt) {
        int t = q[hh ++];
        if (t == k) return dist[k];
        if (t + 1 < N && dist[t + 1] == -1) {
            dist[t + 1] = dist[t] + 1;
            q[++ tt] = t + 1;
        }
        if (t - 1 >= 0 && dist[t - 1] == -1) {
            dist[t - 1] = dist[t] + 1;
            q[++ tt] = t - 1;
        }
        if (t * 2 < N && dist[t * 2] == -1) {
            dist[t * 2] = dist[t] + 1;
            q[++ tt] = t * 2;
        }
    }
    return -1;
}

int main() {
    cin >> n >> k;
    
    cout << bfs() << endl;
    return 0;
}
```





